# Video Generation Post-Training Data Strategy (Cross-Model Comparative Topic) — Anchored on "A Systematic Post-Train Framework for Video Generation" (arXiv:2604.25427), a Cross-Model Survey of SFT Curated-Set Scale/Filtering Criteria and Preference-Pair Annotation Methods

> Topic: Data processing for video generation models (including joint audio-video generation): the data filtering pipeline, data distribution, and annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Filtering Pipeline](#filtering-pipeline) · [Annotation Approach](#annotation-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Video Generation Post-Training Data Strategy (Cross-Model Comparative Topic) — Anchored on "A Systematic Post-Train Framework for Video Generation" (arXiv:2604.25427), a Cross-Model Survey of SFT Curated-Set Scale/Filtering Criteria and Preference-Pair Annotation Methods

### Publishing Organization/Company

Multiple organizations. The anchor paper is a joint work by the University of Hong Kong (HKU; Zeyue Xue, Mengzhao Chen, Ping Luo) + JD Explore Academy (Siming Fu, Jie Huang, Shuai Lu, Haoran Li, Haoyang Huang, Nan Duan, et al.) + Tsinghua University + Peking University + Zhejiang University (Zeyue Xue is also the first author of DanceGRPO; Nan Duan is the corresponding author). The cross-model coverage includes: ByteDance Seed (Seedance 1.0 / 1.5 pro), Tencent Hunyuan (HunyuanVideo / HunyuanVideo 1.5), Kuaishou Kling (Kling 3.0 Omni), Meituan (LongCat-Video), StepFun (Step-Video-T2V), Kunlun Wanwei (SkyReels-V2 / V4), NVIDIA (Cosmos-Predict 2.5), Meta (Movie Gen), Zhipu (CogVideoX), Rhymes AI (Allegro), ByteDance (Goku), Moonshot/Motif (Motif-Video 2B), Sand AI (MAGI-1), Genmo (Mochi 1), Lightricks (LTX-2), OpenAI (Sora 2), Google DeepMind (Veo 3/3.1), ShengShu (Vidu S1), HPC-AI Tech (Open-Sora 2.0), PKU-YuanGroup (Open-Sora Plan), as well as academic-side works such as JavisDiT++, NAVA, ALIVE, and others.

### Publication Date (technical report/paper/open-source release date)

The anchor paper arXiv:2604.25427v1 was submitted on April 28, 2026 (cs.CV, CC BY 4.0). The cross-model objects span from August 2024 (CogVideoX) to the first half of 2026 (Seedance 1.5 pro, Kling 3.0 Omni, SkyReels-V4, Cosmos-Predict 2.5, HunyuanVideo 1.5, LongCat-Video, etc.). The methodological timeline underpinning this work: ImageReward (2023) → VideoReward / Improving Video Generation with Human Feedback (2025-01, arXiv:2501.13918) → DanceGRPO (2025-05, arXiv:2505.07818) → Flow-GRPO (2025-05) → MixGRPO (2025-07) → HPSv3 (ICCV 2025, arXiv:2508.03789) → Self-Forcing (2025-06) → OmniForcing (2026-03), Causal Forcing (2026-02), Astrolabe (2026-03).

### Type (model/dataset/toolchain/benchmark)

A cross-cutting topic entry, not a single model/dataset/toolchain/benchmark. The anchor paper itself is a "methodological technical report" (a practical blueprint / systematic framework); it releases no model weights and no dataset, and functions as an engineering blueprint for a four-phase post-training pipeline. The remaining cross-model objects fall respectively into models (the majority), datasets (HPDv3, the VideoReward preference set), and reward models (HPSv3, VideoAlign/VideoReward, RewardDance, VisionReward).

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced) ⚠️

Anchor paper (2604.25427): the paper itself is open (CC BY 4.0), but neither code nor weights are open-sourced; the base model is "an internal video generation model" (proprietary); neither the SFT dataset nor the RLHF prompt set is open-sourced — the only public visualization is an RLHF effect demo on the publicly available Wan-2.1 in Figure 2. Overall this is "method open, data and model closed."

The cross-model gradient of post-training data openness (from high to low):
① Preference data fully open-sourced: HPDv3 (1.08M text-image pairs, 1.17M pairwise comparison annotations), the VideoReward preference set (16K prompts / 108K videos / 182K annotated triplets, including VideoGen-RewardBench) — these two are currently the most important public preference assets for video/image generation post-training;
② Preference data "about to be released": JavisDiT++'s roughly 25K audio-video preference pairs (not yet public as of this survey) [uncertain];
③ Method and pipeline public, data not public: SkyReels-V2 (30K human-annotated pairs + about 60K DPO data points, 20K per stage across three stages), Step-Video-T2V (the Video-DPO pipeline is fully public, quantity undisclosed), HunyuanVideo 1.5 (the construction of the RLHF prompt set and the GSB annotation protocol are public, scale undisclosed), LongCat-Video (the three-reward GRPO configuration is public, the SFT set size and RM annotation volume are undisclosed), Cosmos-Predict 2.5 (per-domain SFT scale is disclosed item by item across five domains, GRPO configuration is public, but data is not open-sourced — though the post-RL EMA weights are released);
④ Only a single sentence or entirely blank: Sora 2, Veo 3/3.1, LTX-2, Kling 3.0 Omni (only states that DPO was used), Seedance 1.5 pro (only states that RLHF + a multi-dimensional RM was used).

Openness on the reward-model side is markedly better than on the generation-model side: HPSv3, VideoAlign/VideoReward, VisionReward, and the Unified Reward Model all have open-sourced weights, forming the de facto standard combination of "open-source RM + closed-source generator."

### Whether Joint Audio-Video Generation Is Supported, and the Implementation Approach (native joint / cascaded / MoE fusion)

This topic is not a model per se, but the covered objects include many joint audio-video generation models, and their post-training shows a clear stratification:

【Objects that have incorporated AV into post-training rewards】Seedance 1.5 pro — explicitly adopts "an RLHF algorithm customized for audio-video scenarios" together with a multi-dimensional reward model, jointly optimizing motion quality, visual aesthetics, and audio fidelity, and performs infrastructure optimization on the RLHF pipeline that yields nearly a 3× training speedup; Kling 3.0 Omni — samples multiple video variants under the same MVL (multimodal vision-language) condition, and human evaluators compare them to form preference pairs for DPO (though whether audio is scored as an independent dimension is undisclosed); JavisDiT++'s AV-DPO — divides labor across six reward models, where temporal synchrony is handled by Synchformer, audio quality by AudioBox-Aesthetics, and text-audio and cross-modal similarity by ImageBind, making it currently the only work to fully disclose the construction details of AV preference pairs.

【The anchor paper's AV treatment】appears only in the autoregressive distillation phase: for models with audio-video generation capability, it follows OmniForcing (arXiv:2603.11647) in equipping the model with "asymmetric block-causal alignment" and an "audio sink token." That is, AV is reflected only at the level of the distillation architecture; the four reward models used in its SFT and GRPO (video aesthetics / image aesthetics / motion quality / text-video alignment) are all purely visual dimensions, containing no audio or audio-video synchrony reward whatsoever — a notable gap for this framework in the AV era.

【Entirely blank】Academic AV works such as HunyuanVideo-Foley, Ovi (initial version), UniVerse-1, UniTalking, Unison, Foley-Omni, and InstructAV2AV all have no preference-alignment post-training.

### List of Research Sources (URLs of papers/technical reports/official documentation/news, each labeled by nature: official primary / same-team corroboration / third-party report)

- [Official primary] A Systematic Post-Train Framework for Video Generation, arXiv:2604.25427v1, 2026-04-28 (the anchor paper of this entry, HKU + JD Explore Academy + Tsinghua + Peking University + Zhejiang University): https://arxiv.org/abs/2604.25427
- [Official primary] Same as above, full HTML text (including the GRPO formula, isotemporal grouping, Temporal Gradient Rectification, definitions of the four reward models, GSB results 31%/20%): https://arxiv.org/html/2604.25427v1
- [Official primary] DanceGRPO: Unleashing GRPO on Visual Generation, arXiv:2505.07818 (the GRPO foundation for the anchor paper, same first author Zeyue Xue): https://arxiv.org/abs/2505.07818
- [Official primary] Improving Video Generation with Human Feedback (VideoReward/VideoAlign), arXiv:2501.13918 (16K prompts/108K videos/182K triplets/12 T2V models/Bradley-Terry-with-ties modeling/Flow-DPO/Flow-RWR/Flow-NRG): https://arxiv.org/abs/2501.13918
- [Official primary] HPSv3: Towards Wide-Spectrum Human Preference Score, ICCV 2025, arXiv:2508.03789 (HPDv3: 1.08M text-image pairs, 1.17M pairwise comparison annotations): https://arxiv.org/abs/2508.03789
- [Official primary] Flow-GRPO: Training Flow Matching Models via Online RL, arXiv:2505.05470: https://arxiv.org/abs/2505.05470
- [Official primary] MixGRPO: Unlocking Flow-based GRPO Efficiency with Mixed ODE-SDE, arXiv:2507.21802: https://arxiv.org/abs/2507.21802
- [Official primary] RePrompt: Reasoning-Augmented Reprompting for T2I via RL, arXiv:2505.17540 (the paradigm followed by the anchor paper's Prompt Enhancer): https://arxiv.org/abs/2505.17540
- [Official primary] Self Forcing: Bridging the Train-Test Gap in Autoregressive Video Diffusion, arXiv:2506.08009 (the foundation of the anchor paper's AD phase): https://arxiv.org/abs/2506.08009
- [Same-team corroboration] OmniForcing: Unleashing Real-Time Joint Audio-Visual Generation, arXiv:2603.11647 (followed by the anchor paper's AV distillation; authorship substantially overlaps with the anchor paper): https://arxiv.org/abs/2603.11647
- [Same-team corroboration] Astrolabe: Steering Forward-Process RL for Distilled Autoregressive Video Models, arXiv:2603.17051 (same team, Zeyue Xue/Siming Fu/Nan Duan, et al.): https://arxiv.org/abs/2603.17051
- [Official primary] Causal Forcing: Autoregressive Diffusion Distillation Done Right, arXiv:2602.02214: https://arxiv.org/abs/2602.02214
- [Official primary] BranchGRPO: Stable and Efficient GRPO with Structured Branching in Diffusion Models, arXiv:2509.06040: https://arxiv.org/abs/2509.06040
- [Official primary] E-GRPO: High Entropy Steps Drive Effective RL for Flow Models, arXiv:2601.00423: https://arxiv.org/abs/2601.00423
- [Official primary] TempFlow-GRPO: When Timing Matters for GRPO in Flow Models, arXiv:2508.04324: https://arxiv.org/abs/2508.04324
- [Official primary] Coefficients-Preserving Sampling for RL with Flow Matching (Flow-CPS), arXiv:2509.05952: https://arxiv.org/abs/2509.05952
- [Official primary] RewardDance: Reward Scaling in Visual Generation, arXiv:2509.08826: https://arxiv.org/abs/2509.08826
- [Official primary] VisionReward: Fine-grained Multi-dimensional Human Preference Learning, AAAI 2026: https://arxiv.org/abs/2412.21059
- [Official primary] Seedance 1.0: Exploring the Boundaries of Video Generation Models, arXiv:2506.09113 (SFT via targeted collection across hundreds of categories + model merging; multi-dimensional RLHF annotation protocol; three dedicated RMs): https://arxiv.org/abs/2506.09113
- [Official primary] HunyuanVideo: A Systematic Framework for Large Video Generative Models, arXiv:2412.03603 (1 million human-curated SFT samples; a four-item aesthetics + three-item motion rubric): https://arxiv.org/abs/2412.03603
- [Official primary] Kling-Omni Technical Report, arXiv:2512.16776 (multiple variants under the same MVL condition + human comparison forming preference pairs for DPO): https://arxiv.org/abs/2512.16776
- [Official primary] Wan: Open and Advanced Large-Scale Video Generative Models, arXiv:2503.20314: https://arxiv.org/abs/2503.20314
- [Official primary] VBench: Comprehensive Benchmark Suite for Video Generative Models, CVPR 2024: https://arxiv.org/abs/2311.17982
- [Official primary] Qwen3-VL Technical Report, arXiv:2511.21631 (the citation target of the anchor paper's reward-model backbone "Qwen3.5"): https://arxiv.org/abs/2511.21631
- [In-house project research, secondary aggregation] The cross-model figures in this entry (Allegro/Apollo/CogVideoX/Cosmos-Predict25/Goku/HunyuanVideo/JavisDiT_baselines/Kling_30_Omni/LongCat-Video/Movie_Gen/Motif/Open-Sora/Seedance/SkyReels/Step-Video-T2V/Sora_2/Veo_3/caption_models/JAVG_2026_misc, etc.) are all drawn from the `post_training_data` field of each entry under /Users/jan/wangxj/data_survey/video-gen-data-processing/results/; the original provenance can be found in each entry's own sources list

## Data Scale and Distribution

### Training Data Volume (video count/hours/token count, pretraining and SFT reported separately) ⚠️

【Anchor paper】discloses no figures at all: Section 4.1 "Dataset" consists in its entirety of two sentences — "First, we constructed a high-quality text-video dataset for SFT. Subsequently, we curated the prompt set as described in the experimental settings." The SFT set size, RLHF prompt set size, group size N, number of training steps, and GPU scale are all missing. This is the single most important point to flag in this entry: a report titled "a systematic post-training framework" discloses zero figures on the data-scale dimension.

【Cross-model lineage of SFT curated-set scale (publicly comparable figures)】
· Step-Video-T2V: approximately 30 million (30M) high-quality videos;
· Cosmos-Predict 2.5: split across five domains — object permanence 10.4M, driving 3.1M, complex scenes 1.6M, high motion 1.0M, robotic manipulation 730K, plus 4K cooldown data 388K, plus 1.5M driving multi-view segments at 7 cameras, 20 seconds, 30FPS per clip;
· SkyReels-V4: SFT in two stages — 5 million multi-modal-conditioned videos (3 epochs) → 1 million human-curated videos (3 epochs);
· ALIVE: Continue-Training 4.3M → SFT 5M (0.5 epoch) → 1080p Refiner 0.7M → character-specific 0.8M;
· Open-Sora 2.0: Stage 3 5M high-resolution curated set; Open-Sora 1.2: 2M clips / 5k hours; Open-Sora Plan: I2V Stage 2 15M;
· Goku: I2V fine-tuning on 4.5M text-image-video triplets (12.5% of a 36M-video pool);
· Allegro: approximately 2M (0.4% of a 500M raw-clip pool);
· HunyuanVideo original: approximately 1 million human-curated samples; HunyuanVideo 1.5: CT 1 million per task, SFT scale undisclosed;
· CogVideoX: the top-20% highest-quality subset of the pretraining data, 10k steps;
· NAVA: curated 160K from 15M (retention rate ≈1.07%), and re-captioned this subset with the more expensive Gemini-3-Pro;
· Movie Gen: the video SFT set's scale is undisclosed, but "training used only 512 H100s"; the PT2V SFT set is O(1000) samples; the audio SFT set has a cinematic split of O(100)K samples / O(1)K hours + a high-quality-audio split of O(1,000)K samples / O(10)K hours;
· Motif-Video 2B: two SFT passes (480p Stage 7, 720p Stage 10), scale not given.

【Preference data scale lineage】HPDv3: 1.08M text-image pairs / 1.17M pairwise comparisons; VideoReward: 16K prompts / 108K videos / 182K annotated triplets (of which 13K triplets are held out as a validation set whose prompts never appear in the training set); SkyReels-V2: 30K human-annotated pairs used to train a Bradley-Terry reward model + approximately 60K DPO data points (20K per stage × 3 stages); JavisDiT++: approximately 25K audio-video preference pairs (prompt pool of 30K, 1 epoch, LoRA with 121M trainable parameters); HunyuanVideo 1.5's T2V-side RLHF prompt set is on the order of O(10K)-tens-of-thousands.

【Scale regularities】The typical scale of SFT curated sets is 10^6–10^7 samples, at 0.4%–20% of the pretraining pool; the typical scale of preference data is 10^4–10^5 pairs, 2–3 orders of magnitude smaller than the SFT set — consistent with the proportional structure seen in LLM post-training.

### Data Source Composition (proprietary/public datasets/web crawl/licensed procurement/synthetic data) ⚠️

【Anchor paper】only states that the SFT set is a "high-quality text-video dataset" and RLHF uses a "curated prompt set"; the source composition is entirely unspecified. Since the base model is proprietary, the SFT data is very likely drawn from a high-scoring subset of its pretraining corpus, but there is no textual evidence for this [uncertain].

【Four paradigms of post-training data sourcing, cross-model】
① High-scoring subsets within the pretraining pool (most common): CogVideoX (top 20%), Allegro (500M→2M), Motif, Open-Sora, MAGI-1 (more stringent filtering in the final stage), NAVA (15M→160K) — essentially "same-source purification";
② Targeted collection / independently procured premium sets: Seedance 1.0 defines "hundreds of categories" by attributes such as visual style and motion type for targeted collection; Movie Gen has humans hand-pick "cinematic" footage and rewrite captions; LongCat-Video incorporates additional dedicated datasets for camera motion and visual style;
③ Model-self-generated data (candidate source for RLHF/DPO): Step-Video-T2V generates multiple videos per prompt with different random seeds; Kling 3.0 Omni samples multiple variants under the same MVL condition; JavisDiT++ generates N=3 candidates per prompt plus 1 ground truth, totaling 4; Seedance 1.0's preference data "covers synthetic videos generated at different stages of the model, among other sources"; StreamChar's Stage II distillation directly uses online rollouts from the student model;
④ Feedback loop of real online user prompts: Seedance 1.0 explicitly states it "collects prompts from the training set and from online users, performing data balancing and information filtering to remove duplicate and ambiguous prompts" — a structural advantage of closed-source commercial models relative to academic work.

【The necessity of de-duplication between the prompt set and the SFT set】JavisDiT++ explicitly states its 30K-prompt DPO pool "does not overlap with the SFT training data (apart from the SFT training data)"; VideoReward keeps 13K prompts that never appear in the training set as triplets for a validation set. Together, these show that prompt leakage during post-training is a risk that is explicitly guarded against.

### Data Compliance and Provenance (share of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Neither the anchor paper nor the vast majority of cross-model objects disclose anything on the copyright and provenance dimension of post-training data [uncertain]. Relevant facts that can be recorded:
· Because post-training data is small in scale (10^4–10^7), it is in principle easier to achieve rights-clearance than pretraining data. Movie Gen's "manually hand-picked cinematic footage + manually rewritten captions" pipeline and SkyReels-V4's "1 million human-curated samples" both have a cost structure that would support per-item licensing review, but neither states that this was done;
· The additional compliance dimension for preference data is "annotator labor" — Step-Video-T2V mentions that preference annotation was "supervised throughout by quality-control staff for consistency," HunyuanVideo 1.5 uses GSB annotation, and Seedance 1.0 uses a multi-dimensional annotation protocol of "selecting the best/worst along a specified dimension while ensuring the best sample is not inferior to the worst along other dimensions" — but none disclose the annotation team's size, sourcing, compensation, or training;
· Model-self-generated preference candidates (Seedance 1.0, Step-Video, Kling, JavisDiT++) sidestep external-material copyright issues but introduce a "synthetic data feedback" distribution risk;
· The publicly available preference datasets HPDv3 (1.08M text-image pairs) and VideoReward (108K videos sourced from 12 T2V models: Gen2, SVD, Pika 1.0, Vega, PixVerse v1/v2, HiDream, Dreamina, Luma, Gen3, Kling 1.0/1.5) — the latter's videos are entirely generated by commercial models, and the licensing status of their redistribution is an unaddressed gray area. [uncertain]

### Clip Duration Distribution and Segmentation Strategy ⚠️

【Anchor paper】discloses nothing about the SFT data's duration distribution or segmentation strategy [uncertain]. Only from the autoregressive distillation (Self-Forcing + frame-by-frame rollout with KV cache) can one infer that its deployment target is streaming long-form video.

【Cross-model comparable figures】Movie Gen's video SFT set has durations of 10.6–16 seconds, with 50% at 16 seconds (markedly long, favoring complete narratives); Allegro's SFT strictly bounds duration to 6–16 seconds; MAGI-1 relaxes to ≤16s in its final stage; both LongCat-Video's SFT and GRPO use 93 frames; Cosmos-Predict 2.5's driving multi-view SFT uses 20 seconds at 30FPS.

【Regularity】Post-training generally pushes the duration ceiling toward the limit of the model's capability (around 16 seconds) and favors longer clips, in contrast to the pretraining stage's preference for short clips (2–8 seconds) — because SFT aims to teach "the narrative and motion completeness of a full shot," while the RLHF stage, given the extremely high cost of rollout (the anchor paper explicitly states "rollout generation is expensive"), instead tends toward shorter clips.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

【Anchor paper】undisclosed [uncertain].

【Cross-model】Post-training almost uniformly adopts "the final rung of the resolution ladder": CogVideoX SFT is 768×1360; Allegro SFT requires ≥1280×720; LongCat-Video's SFT and GRPO are both trained jointly at 480p+720p; Motif-Video 2B sets two SFT passes at 480p and 720p, and its 720p pretraining starts from the 480p SFT checkpoint (rather than from the pretraining checkpoint) — an unconventional move that inserts SFT into the middle of the resolution ladder; SkyReels-V2 first performs 540p concept-balancing SFT, then 720p high-quality SFT; HunyuanVideo 1.5's CT stage is 480p/720p at 24fps, with an additional 1 million 1K–4K clips used to separately train a super-resolution module; ALIVE uses 0.7M high-clarity samples to train a dedicated 1080p Refiner.

【Regularity】SFT is the closing stage of the resolution curriculum, while RLHF/GRPO, due to rollout cost, is usually conducted at a resolution equal to or lower than SFT's (LongCat keeps 480p+720p; Cosmos uses a minimal configuration of 8 rollouts × 20 diffusion steps). Aspect-ratio bucketing is generally no longer separately discussed at the post-training stage. [uncertain]

### Category/Domain Distribution and Mixture Strategy (proportion control and concept balancing across characters, actions, scenes, styles, etc.) ⚠️

This is the dimension with the greatest differentiation in post-training data strategy, and the one that most reveals a team's engineering maturity. The anchor paper is entirely silent here (only stating "curated the prompt set") [uncertain], but four clearly distinguishable mixture paradigms exist across the cross-model objects:

【Paradigm 1: inverse-density sampling for long-tail uplift (most elegant)】LongCat-Video's SFT curated set, on top of multi-metric filtering (aesthetic score / video quality / motion quality), applies a second layer of sampling proportional to the inverse of a sample's density in the caption-embedding space (inversely proportional to their density in the caption embedding space), directly implementing a relative uplift of long-tail concepts. This represents formalizing "concept balancing" into a computable criterion.

【Paradigm 2: k-NN concept balancing + manual cinematic-quality curation】Movie Gen's video SFT set is produced through four stages: strict automated threshold filtering → k-NN concept balancing → manual selection for "cinematic" quality → manual caption rewriting; the paper explicitly notes that "several million" samples remain after the first stage but are concept-imbalanced, i.e., concept balancing is a separate step from quality filtering.

【Paradigm 3: classifier-based domain split + independent per-domain SFT (the Physical AI route)】Cosmos-Predict 2.5 trains a multi-head classifier on InternVideo2 embeddings to sort samples into five domains, trains an independent SFT model per domain (30k iterations, batch 256 each), and then performs model merging. The scale differs enormously across domains (object permanence 10.4M vs. robotic manipulation 730K), reflecting the priority placed on the basic physical common sense that "objects do not vanish when occluded." Per-domain human win rates are all significantly better than the pretraining baseline: robotic manipulation 72.6% vs. 8.3%, object permanence 50.9% vs. 27.7%, high motion 44.0% vs. 34.7%, complex scenes 42.6% vs. 35.4%, driving 47.9% vs. 28.8%.

【Paradigm 4: targeted collection across hundreds of categories + sub-model fusion】Seedance 1.0 defines "hundreds of categories" by attributes such as visual style and motion type for targeted collection, trains multiple sub-models each covering different styles/motions/scenes, and then performs model merging, using a smaller learning rate than pretraining, a limited GPU count, and early stopping to prevent overfitting and preserve text controllability. This is the idea of "replacing a single mixed SFT set with the weight-average of multiple specialized SFT models," sharing its lineage with Movie Gen's model averaging.

【Mixture on the preference-data side】HunyuanVideo 1.5's T2V RLHF prompt set is category-balanced along three dimensions — motion / scene / subject — sourced from a mix of LLM-generated prompts and training captions; on the I2V side it builds a prompt set covering 100+ categories, with accompanying images hand-picked from high-aesthetic images and manually verified for text-image consistency. Seedance 1.0's RLHF prompts undergo "data balancing and information filtering" to remove duplicate and ambiguous prompts.

【Motif-Video 2B's iterative gap-filling】Its SFT corpus assembly uses iterative gap-filling: on top of routine filtering, it layers a stricter aesthetic cutoff, style/subject-tag-driven domain-balancing, and, on the video side, an action=Dynamic admission criterion for motion.

### Audio Category Distribution and Mixture (proportion and control strategy for speech/foley/music/ambient sound/silence) — an AV-model-specific dimension ⚠️

None of the anchor paper's four reward models (video aesthetics, image aesthetics, motion quality, text-video alignment) touch on audio, and the SFT data's audio-category mixture is likewise unspecified — a notable gap for this framework relative to the AV mainstream of 2026 [uncertain].

【Cross-model: only three examples give an audio-side post-training mixture】
① Movie Gen's audio SFT set is the most finely disclosed in this topic: two splits — the cinematic split (professionally produced, containing both in-scene diegetic audio and off-screen ambient/thematic music, explicitly excluding clips containing human vocals, auto-filtered by a cinematic-quality classifier + AED sound-event detection and then manually selected) at O(100)K samples / O(1)K hours; the high-quality-audio split (video-free high-quality music, O(10)K hours + sound effects, O(10)K hours) at O(1,000)K samples / O(10)K hours. "Excluding vocal-containing clips" is a key decision: Movie Gen Audio is positioned as a sound-effects-and-music generator rather than a dialogue generator, and therefore actively removes speech-type samples during the SFT stage.
② Foley-Omni's Stage 3 (full V2ST soundtrack fine-tuning) uses the smallest-scale data — 216 hours — for a lightweight 2-epoch fine-tune, paired with 100 hours of replay per domain to prevent forgetting; the admission criterion is "contains multiple audio components";
③ JavisDiT++'s AV-DPO uses AudioBox-Aesthetics to independently assess audio quality and Synchformer to independently assess temporal synchrony — i.e., audio is treated as an independent modality at the preference-ranking level — but the ratio of speech/sound-effects/music is undisclosed.

【Seedance 1.5 pro】states its multi-dimensional reward model covers "audio fidelity," but whether speech/sound-effects/music are scored along separate dimensions is undisclosed [uncertain]. Whether Kling 3.0 Omni's audio-dimension preference annotation (lip-sync degree, timbre consistency) is scored as an independent dimension is likewise undisclosed [uncertain].

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

The anchor paper does not address this [uncertain]. Cross-model facts worth recording:
· 50% of Movie Gen's video SFT set is 16-second-long clips, and the manual selection criterion of "cinematic" quality implies a predominance of complete single-shot narratives;
· Post-training generally favors single-shot, transition-free footage — one of Step-Video-T2V's human-review criteria is explicitly "whether the scene transition is smooth," and Motif's SFT admission includes action=Dynamic;
· Multi-shot capability is mainly handled by pretraining and by prompt structuring at inference time; no work has been observed to explicitly control shot-count distribution at the SFT/preference-data level [uncertain];
· Autoregressive distillation (the anchor paper's Phase 4, StreamChar, LongLive, OmniForcing, Causal Forcing) turns the narrative-structure problem into a "long-horizon rollout error accumulation" problem, whose training data is the student model's own self-generated online rollouts rather than collected footage — a technical route entirely distinct from data collection on the narrative-structure dimension.

### Language/Accent Distribution (the data basis for multilingual lip-sync capability) ⚠️

Neither the anchor paper nor the vast majority of cross-model objects disclose language/accent mixture at the post-training stage [uncertain]. Indirect facts worth recording:
· The anchor paper's prompt enhancer (Phase 3) is an LLM trained with GRPO; the language composition of its training prompts is unspecified [uncertain];
· MOVA builds a 732-item Chinese-English bilingual arena evaluation set (for evaluation, not training); UniTalking's 20-person × 50-item blind test and Unison's 40-sample × 25-person ranking vote are likewise for evaluation only;
· Both public preference sets, HPDv3 and VideoReward, are predominantly English-prompted [uncertain]; preference data for Chinese-language scenarios is a clear gap in the public ecosystem;
· HunyuanVideo 1.5's RLHF prompt set is sourced from "a mix of LLM-generated prompts and training captions," with language composition unspecified [uncertain].

## Filtering Pipeline

### Overall Structure of the Filtering Funnel (number of filtering stages, ordering) ⚠️

【The anchor paper's four-phase post-training pipeline (the skeleton of this entry)】Phase 1 SFT (using curated data to establish a stable instruction-following baseline and reference policy) → Phase 2 RLHF (a GRPO-based trainer that aligns aesthetics, motion quality, and text alignment via multi-dimensional rewards) → Phase 3 PE prompt enhancement (using the same reward loop, GRPO trains an LLM to rewrite user inputs) → Phase 4 AD autoregressive distillation (a self-forcing objective that transfers capability to a causal architecture to improve inference efficiency).

The paper's core thesis is "SFT as the foundation for RLHF": SFT's goal is not to solve alignment or optimize subjective quality, but to establish a stable, well-structured reference policy that eliminates the most severe and frequent failure modes and prevents subsequent RL from diverging into degenerate behavior, and moreover "SFT also enlarges the exploration for RLHF." The second thesis is "PE complements RLHF": RLHF optimizes the output-side generation policy, PE optimizes the input-side prompt, and both are trained with the same reward set (human preference, visual realism, semantic alignment), forming dual-sided input-output alignment.

【Important quality caveat】The specific wording of Section 3.1 on SFT is clearly migrated from LLM post-training text — the failure modes listed are "refusal cascades, incoherent reasoning, unsafe outputs," which are language-model failure modes rather than video-diffusion-model failure modes (the video-side equivalents should be things like hand errors, text-rendering errors, and rapid-motion collapse, which are in fact mentioned in the paper's own introduction). This paragraph contains no video-specific data-construction detail, and readers should not treat it as a reproducible SFT data methodology.

【Cross-model lineage of post-training pipeline shapes】
· Four-stage (SFT→RLHF→PE→distillation): the anchor paper, Seedance 1.5 pro (SFT → AV-customized RLHF), Kling 3.0 Omni (quality-tuning SFT → DPO);
· Three-stage CT→SFT→RLHF: HunyuanVideo 1.5 (with T2V and I2V kept entirely separate throughout);
· Two-stage SFT→DPO: Step-Video-T2V (Step-3 SFT → Step-4 Video-DPO), SkyReels-V2 (concept-balancing SFT → high-quality SFT → three-stage DPO);
· Two-stage SFT→GRPO: LongCat-Video, Cosmos-Predict 2.5 (five-domain SFT → model merging → GRPO);
· SFT only, no preference learning (the vast majority on the open-source/academic side): Movie Gen, CogVideoX, Allegro, Goku, Motif, MAGI-1, the Open-Sora series, NAVA, ALIVE, Apollo;
· No post-training at all: MOVA (folds SFT into the tail end of the pretraining curriculum), Unison, UniTalking, UniVerse-1, HunyuanVideo-Foley, Foley-Omni, CineDance, Mochi 1;
· Using inference-time search in place of post-training: ITS-JAVG (multiple verifiers + Best-of-N/EvoSearch, JavisDiT 5 samples, MMDisCo 10 samples), arguing that comparable results can be reached without post-training at all.

### Quantitative Funnel Retention Rates (input/output volume at each filtering stage and the final retention rate, e.g., Apollo 27%) ⚠️

The anchor paper gives no quantitative funnel data whatsoever [uncertain]. The comparable "post-training retention rates" (SFT curated set ÷ pretraining pool) across the cross-model objects are as follows — the single most valuable set of comparable numbers in this topic:
· NAVA: 160K ÷ 15M ≈ 1.07% (the strictest in this survey, "one in a thousand kept");
· Allegro: 2M ÷ 500M = 0.4% (even stricter if measured against raw clips, but its 2M is the direct output of the strictest combination of thresholds);
· Goku I2V: 4.5M ÷ 36M = 12.5% (with the filtering criterion emphasizing domain diversity rather than a quality score);
· CogVideoX: top 20% (explicitly defined by percentile rather than a fixed threshold);
· Open-Sora Plan T2V Stage 3: after filtering, approximately 19M / 27% of Panda70M is retained;
· NVIDIA Cosmos WFM: roughly 10^7 clips cut for fine-tuning from a pretraining pool of roughly 10^8 (about 10%, filtering criteria undisclosed);
· SkyReels-V4: 5 million → 1 million human-curated (the second SFT stage's retention rate is 20%).

【Empirical range】The retention rate of SFT curated sets relative to the pretraining pool clusters between 0.4% and 20%, with a median of roughly 5%–10%; models chasing aesthetics and "cinematic" quality tend toward lower retention (Allegro, NAVA), while models chasing domain coverage or multi-task capability tend toward higher retention (Goku, Open-Sora Plan, Cosmos).

【The "retention rate" on the preference-data side takes a different form】Its scale is determined by annotation budget rather than a filtering threshold — SkyReels-V2's 30K human-annotated pairs, Step-Video's undisclosed count, and HunyuanVideo 1.5's O(10K) all reflect a "ceiling on annotation capacity." JavisDiT++ gives a distinctive metric: about 30% of the "winning" samples in its final preference data come from model generation rather than ground truth, from which the authors infer "the baseline model itself already possesses fairly strong generation capability" — this ratio can serve as an empirical signal for judging whether a model has reached the stage where "self-generated data can be used for preference optimization."

### Shot Segmentation Method (PySceneDetect / in-house model / shot-aware splitting) ⚠️

Not addressed by the anchor paper [uncertain]. Post-training generally does not redo shot segmentation — the SFT curated set is a filtered subset drawn from a pretraining pool that has already been segmented, so shot segmentation is the responsibility of the upstream pretraining data pipeline. The one relevant cross-model fact is that Step-Video-T2V's SFT human review lists "whether the scene transition is smooth" as one of four criteria, i.e., using human review as a last line of defense against residual segmentation problems (a transition mixed into a single clip) at the post-training stage — one of the few pieces of evidence for "post-training as the final safeguard on segmentation quality." Motif-Video 2B's SFT admission criterion of action=Dynamic indirectly excludes static transition remnants.

### Quality Filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

【Anchor paper】The quality-filtering standard for SFT data is entirely undisclosed [uncertain]. Quality control is handed off entirely to the four reward models of the RLHF stage (video aesthetics: lighting, composition, color harmony, temporal consistency, cross-frame cinematic quality; image aesthetics: frame-level perceptual quality, sharp detail, pleasing structure; motion quality: motion realism, smoothness, coherence, suppressing shaky/discontinuous motion and temporally inconsistent object transitions; text-video alignment: semantic consistency). This reflects a paradigm shift — moving "what counts as high quality" from a data-filtering threshold into a learnable reward model.

【SFT quality-filtering criteria across models (the few cases with threshold-level disclosure)】
· Allegro (the most complete set of thresholds): duration 6–16 seconds, resolution ≥1280×720, brightness [20,180], DOVER ≥0.07, LPIPS ≥0.05, UniMatch motion [1.0,100], aesthetics ≥5.3, text area ≤0.05%, CLIP similarity ≥0.20;
· HunyuanVideo original: 1 million human-curated samples, criteria being a four-item aesthetics rubric (color harmony, lighting, subject prominence, spatial composition) + a three-item motion rubric (motion speed, action completeness, motion blur) — this seven-dimensional human rubric is the most widely cited SFT filtering rubric in this topic;
· HunyuanVideo 1.5 SFT: on top of the CT data, a stricter filter is applied along three items — aesthetic appeal, sharpness, motion fluency — before final construction via human annotation;
· CogVideoX: filters out the top 20% against three categories of dirty data ("subtitles, watermarks, low bitrate");
· Step-Video-T2V: a three-step process — automatic scoring and heuristic-rule filtering → VideoCLIP K-means intra-cluster outlier removal (samples beyond a distance threshold from the cluster center are removed, threshold undisclosed) → item-by-item human review (sharpness, aesthetics, whether the motion is appropriate, whether the scene transition is smooth) with caption refinement;
· LongCat-Video: a first layer of multi-metric filtering (aesthetic score, video quality, motion quality) + a second layer of inverse-density sampling in caption-embedding space;
· Motif-Video 2B: routine filtering + a stricter aesthetic cutoff + style/subject-tag-driven domain-balancing + action=Dynamic;
· NAVA: multi-operator collaborative filtering, with a criterion of "accurate captions + strong audio-visual alignment," threshold undisclosed.

【Commonality】SFT filtering = pretraining thresholds + significantly raised aesthetic/sharpness bars + concept balancing + final human review. Final human review appears in almost all industrial-grade works — HunyuanVideo (both the original and 1.5), Step-Video-T2V, Movie Gen, SkyReels-V4, Apollo Stage III — making it the hallmark step that distinguishes SFT filtering from pretraining filtering.

### Motion Filtering (optical-flow/motion-score thresholds, removal of static or shaky footage) ⚠️

The anchor paper has no data-side description of motion filtering, but on the reward side it has a dedicated Motion Quality reward model (assessing the realism, smoothness, and coherence of motion dynamics, suppressing shaky footage, discontinuous motion, and temporally inconsistent object transitions), and its results report motion quality as one of the two dimensions with the most significant RLHF gains [data-side uncertain].

【Cross-model】
· Allegro SFT: UniMatch optical-flow motion score range [1.0,100];
· HunyuanVideo original SFT: human criteria include motion speed, action completeness, and motion blur;
· Motif-Video 2B SFT: explicitly requires action=Dynamic;
· SkyReels-V2's RL objective explicitly focuses on motion quality (dynamic consistency and fluency) rather than general aesthetic preference, and the automated side of its preference data is precisely "corrupted samples produced by applying controlled distortion to real videos";
· LongCat-Video's Motion Quality reward model has a design worth noting: fine-tuned on internally annotated data starting from a VideoAlign backbone, with grayscale video as input (color removed to force the model to evaluate only motion without being distracted by color/aesthetics) — a direct engineering technique for decoupling motion reward from aesthetic reward;
· Cosmos-Predict 2.5 sets aside a dedicated "high motion" domain (1.0M samples) for domain-specific SFT.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Not addressed by the anchor paper [uncertain]. The deduplication focus at the post-training stage differs from pretraining: it is not about removing duplicate footage but about removing duplicate/ambiguous prompts. Seedance 1.0 explicitly performs "data balancing and information filtering to remove duplicate and ambiguous prompts" after RLHF prompt collection; Step-Video-T2V has annotators synthesize supplementary prompts per guidelines to ensure prompt diversity; JavisDiT++ ensures its 30K-prompt DPO pool does not overlap with the SFT data; VideoReward keeps 13K prompts that never appear in the training set as triplets for a validation set.

The most typical form of material-level deduplication at the SFT stage is Step-Video-T2V's VideoCLIP K-means intra-cluster outlier removal (formally a clustering operation, functionally serving both semantic deduplication and outlier removal) and LongCat-Video's inverse-density sampling in caption-embedding space (down-sampling high-density, i.e., near-duplicate, regions — a continuous, soft form of deduplication). Both represent a direction of "replacing hard deduplication thresholds with embedding-space density."

### VLM/LLM as Quality Inspector (multimodal large-model quality scoring and mismatch removal — the 2026 trend from shallow scorers toward large-model semantic judgment)

This is the core of the topic — at the post-training stage, "model as judge" is no longer merely a data filter but becomes the training signal itself (the reward model), the single most important paradigm shift of 2025–2026.

【The anchor paper's reward-model system (the relatively well-disclosed part)】Following the HPSv3 training paradigm, it uses Qwen3.5 (a citation pointing to the Qwen3-VL technical report, arXiv:2511.21631) as a backbone to extract features from image and text, with an MLP head outputting a scalar score; for a training image pair (x1,x2), text c, and human preference annotation (y1,y2), it computes r1 and r2, adopting an "uncertainty-aware ranking loss"; training is split into two stages — Stage 1 uses "data-aware orthogonal gradient projection" to integrate the diverse aesthetic preferences from HPDv3++ while retaining the original human-preference knowledge already encoded in HPSv3; Stage 2 further leverages "unlabeled data produced by models of different capability levels and different RL iterations." The final result is four reward metrics covering video aesthetics, text-video alignment, image aesthetics, and text-image alignment.

【Engineering difficulty of multi-reward fusion (explicitly identified by the paper)】Different reward signals differ in granularity, scale, and optimization tendency: emphasizing text-video alignment improves semantic fidelity but sometimes harms visual naturalness; over-prioritizing motion quality or video aesthetics leads to visually pleasing but semantically weaker output. Careful design of a reward-aggregation strategy and weighting coefficients is therefore required so that optimization remains stable and is not dominated by any single objective. The paper states its own final tradeoff prioritizes "overall visual quality."

【Cross-model reward-model inventory】
· VideoAlign / VideoReward (arXiv:2501.13918, Kuaishou Kling + CUHK): VLM-based, three dimensions (visual quality VQ / motion quality MQ / text alignment TA), trained from 16K prompts, 108K videos generated by 12 T2V models, and 182K annotated triplets, using a Bradley-Terry model with ties (BTT). Widely reused as a backbone by Cosmos-Predict 2.5, LongCat-Video, and JavisDiT++, and the de facto standard open-source video reward model;
· HPSv3 (ICCV 2025): HPDv3 contains 1.08M text-image pairs and 1.17M pairwise comparison annotations, covering SOTA generative models as well as low-to-high-quality real images;
· LongCat-Video's three rewards: VQ = HPSv3-general (average over all frames) + HPSv3-percentile (computed on the top 30% highest-scoring frames combined with the video caption, used to suppress the dilution of overall judgment by a minority of low-quality frames) — a dual-path design; MQ and TA are both fine-tuned on internally annotated data starting from a VideoAlign backbone;
· Seedance 1.0's three dedicated RMs: Foundational RM (VLM architecture, text-image alignment and structural stability), Motion RM (suppressing artifacts, enhancing motion magnitude and liveliness), Aesthetic RM (image-space input, with the data source switched to video keyframes), with multi-round iterative learning;
· SkyReels-V2: a Bradley-Terry model trained on 30K sample pairs;
· JavisDiT++'s six-model division of labor: AudioBox-Aesthetics (audio quality), ImageBind (text-audio alignment / text-video alignment / cross-modal similarity), VideoAlign (video quality), Synchformer (temporal synchrony);
· Other cited reward-model lineages: ImageReward, Pick-a-Pic, VideoScore, VisionReward (AAAI 2026), Unified Reward Model, RewardDance.

【Counter-example and cautionary note】Step-Video-T2V explicitly does not train a reward model, and in its outlook notes a limitation of current DPO — gains saturate once the model can easily distinguish positive from negative samples — proposing that future work introduce an RM to dynamically score newly generated samples so as to keep providing an effective gradient. The verifier-hacking problem revealed by ITS-JAVG foreshadows: recklessly using automatic verifiers to construct preference data for RLHF is likely to train a model that merely flatters the judge. Cosmos-Predict 2.5's response is to regularize with a diffusion loss on the fine-tuning dataset to mitigate reward hacking; LongCat-Video's and the anchor paper's response is to use multiple rewards jointly so they check one another.

### Safety and Compliance Filtering (NSFW, copyright, faces/privacy) ⚠️

The anchor paper's Section 3.1 lists "unsafe outputs" as one of the failure modes to be eliminated at the SFT stage, but gives no specific filtering method, classifier, or data-processing description [uncertain] (and, as noted above, this wording is suspected to be migrated from LLM post-training text). Section 6's Broader Impact discusses commercial application value rather than risk mitigation.

【Cross-model】Sora 2 is the only object with detailed post-training-dimension safety disclosure, but its content is safety-alignment evaluation rather than capability post-training: through targeted red-teaming it collected "thousands of adversarial prompts," classified by use case and policy area, generated outputs with a helpful-only version of the video model, and scored them to convert into an automated evaluation set measuring the two metrics not_unsafe and not_overrefuse (adult nudity/sexual content without portrait 96.04%/96.20%, with portrait 98.40%/97.60%; self-harm 99.70%/94.60%; violence and gore 95.10%/97.00%; prohibited political persuasion 95.52%/98.67%; extremism and hate 96.82%/99.11%). What Veo 3/3.1's official documentation calls "post-training mitigations" in fact refers to SynthID watermarking and production-environment output filtering — deployment-side measures rather than data-side. InstructAV2AV's Qwen3-Omni five-dimensional automatic verifier includes "safety" as one of the criteria for training-sample admission, a rare instance on the academic side of folding safety into SFT admission.

【Overall assessment】In public materials, post-training safety work is almost entirely concentrated on "output-side interception + red-team evaluation," rather than "constructing safety preference pairs within preference data." Using RLHF for safety alignment (the standard LLM-domain approach) has not yet been observed in public practice in the video-generation domain. [uncertain]

## Annotation Approach

### Caption Model Used (in-house VLM / open-source model, model scale) ⚠️

The anchor paper does not disclose the caption model used for SFT data [uncertain], but its Phase 3 prompt enhancement (PE) is in essence "using GRPO to train an LLM to rewrite prompts," adopting the RePrompt (arXiv:2505.17540) paradigm — the generation backbone is frozen and only the policy πθ is optimized, so it can be plugged seamlessly into any off-the-shelf T2I/T2V generator. PE's model scale, backbone name, and training-prompt scale are all undisclosed [uncertain].

【Cross-model caption/prompt model practices at the post-training stage】
· NAVA's approach is the most noteworthy: the caption for its SFT curated subset (160K) is regenerated by the more expensive Gemini-3-Pro (the original pretraining caption was generated by Flash), i.e., a dual purification of "curated subset + upgraded annotator" — the SFT stage not only swaps the data but also swaps the annotator;
· Open-Sora 2.0: the Stage 3 5M curated dataset is re-captioned with Qwen2.5-Max;
· Movie Gen: the SFT set's captions are manually rewritten (the last of the four stages);
· Step-Video-T2V: the SFT human-review step simultaneously refines captions;
· MAGI-1's inference-side Prompt Enhancement small model (about 7B) is distilled from roughly 2 million MLLM-generated enhanced prompt corpora, filtering out samples whose target text is too long to control output length — the only publicly disclosed figure for PE training-data scale;
· Open-Sora Plan's prompt refiner: 19,500 captions, LLaMA-3.1-8B LoRA (rank 64, batch 32, 1 epoch, 30 minutes on a single H100) — the cost lower-bound example for PE;
· MOVA replaces post-training instruction alignment with an inference-side prompt rewriter (Qwen3-VL extracts structured visual descriptions + Gemini 2.5 Pro rewrites via in-context learning into prompts matching the training distribution), raising human Arena ELO from 982.9 to 1025.3 — a quantitative demonstration of PE's independent value.

【Regularity】Re-captioning is a standard, non-optional step at the SFT stage: pretraining uses a cheap model for bulk captioning, while the SFT subset is re-captioned with the most expensive model available or by hand — currently the recognized cost-optimal configuration.

### Caption Density and Structuring (short/long/dense description, structured fields such as camera motion and style tags) ⚠️

The anchor paper does not disclose the structure or density of SFT captions [uncertain]. PE's reward design includes a "Structure Reward," enforcing format compliance and length constraints to ensure the prompt is valid and executable — the only piece of information related to caption structure, indicating that the enhanced prompt follows an explicit structured template, though the template's content is undisclosed [uncertain].

【Cross-model】The trend in caption structure at the post-training stage is "longer, denser, more structured": NAVA's SFT captions, generated by Gemini-3-Pro, are "more accurate, more structured, temporally grounded" audio-visual descriptions; HunyuanVideo 1.5's RLHF prompt set is sourced from a mix of LLM-generated prompts and training captions, indicating that the distributional gap between training captions and user prompts is explicitly modeled; MOVA's rewriter is explicitly aimed at "bridging the gap between user input and the training data distribution" — this actually reveals the core tension in SFT caption-structure design: denser captions are more beneficial for training, but drift further from the distribution of actual user prompts, so PE must be paired with them to realize their benefit. The anchor paper's decision to list PE as a separate phase is precisely a systematic response to this tension.

### Joint Audio-Video Caption Structure (whether visual + auditory tracks are covered simultaneously, whether split into separate fields — e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields) ⚠️

The anchor paper does not touch on joint audio-video captioning at all [uncertain] — its four reward models are all visual dimensions, and PE's three rewards (text-video alignment, video aesthetics, structure reward) likewise have no audio item. For models with AV generation capability, the paper only cites, at the distillation phase, OmniForcing's asymmetric block-causal alignment and audio sink token — an architectural rather than annotation-schema matter.

【Cross-model AV captioning practices at the post-training stage】
· NAVA's re-captioned SFT subset via Gemini-3-Pro produces precisely "more accurate, structured, temporally grounded audio-visual captions" — a clear case of upgrading joint AV annotation at the post-training stage;
· Movie Gen's audio SFT set's cinematic split is auto-filtered by a cinematic-quality classifier + AED (sound-event detection) and then manually selected, with the annotation scheme distinguishing in-scene diegetic audio from off-screen ambient/thematic music;
· JavisDiT++'s preference ranking scores separately along the text-audio and text-video alignment chains via ImageBind — equivalent to splitting the caption into two separate audiovisual evaluation tracks;
· Post-training for captioners on the ecosystem side is already highly mature: AVoCaDO's GRPO reward includes a five-dimension checklist coverage score (judged by GPT-4.1), a dialogue F1 (speaker accuracy + content edit-distance DP-alignment similarity, threshold 0.6), and a length regularizer (penalty above 4096 tokens); AVSCap's mixed GRPO reward = length control + speech fidelity + audio-visual consistency, with the key finding that "RL gains > expanding SFT data volume"; Tencent Hunyuan 1.5's caption model uses OPA-DPO to suppress multimodal hallucination. These are mature post-training paradigms on the captioner side, but have not yet propagated back into the AV preference-reward design of the video-generation models themselves.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Entirely unaddressed by the anchor paper [uncertain]. Cross-model: dialogue and speaker-attribute annotation is almost entirely absent at the post-training stage — Movie Gen's audio SFT cinematic split, on the contrary, explicitly excludes clips containing human vocals; Seedance 1.5 pro states its reward covers audio fidelity but does not say whether it includes dialogue accuracy or speaker consistency [uncertain]; whether Kling 3.0 Omni's preference annotation includes lip-sync degree or timbre consistency is undisclosed [uncertain].

The transferable experience from the ecosystem side comes from captioner post-training: AVoCaDO's dialogue-F1 reward (speaker accuracy + content edit-distance DP-alignment similarity, threshold 0.6) is currently the only design that formalizes "dialogue transcription accuracy" as an optimizable reward, which in principle could be transferred directly as a dialogue-dimension reward for AV generation models, but as of this survey no generation-side work has done so.

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action labels, explicit state) ⚠️

Not addressed by the anchor paper [uncertain]. Only two cross-model examples exist: Cosmos-Predict 2.5's domain-specialized post-training uses a "world scenario map" (projected from HD maps and 3D boxes) as the control signal for the Cosmos-Transfer2.5 driving ControlNet — the clearest case of using geometric structured annotation at the post-training stage; LTX-2 has released a series of control LoRAs for camera motion, pose control, and lip dubbing, which clearly require specially constructed geometric/pose paired datasets, but the construction method is entirely undisclosed [uncertain]. LongCat-Video incorporates a dedicated dataset on "camera motion and visual style" at the SFT stage to strengthen instruction following, but does not state whether it includes explicit camera-parameter annotation [uncertain]. LongCat Avatar 1.5 introduces a first-frame hand-visibility check in GRPO to prioritize sampling hand-containing samples — a distinctive use of a structured detector for rollout sampling scheduling.

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV)

There are two entirely different paths for "synthetic data" at the post-training stage, which must be strictly distinguished in this entry:

【Path 1: constructing negative samples via controlled distortion (automated preference pairs)】SkyReels-V2's "semi-automatic data production line" is the most complete public case — the automated side applies controlled distortion to real video to generate corrupted samples, which naturally form preference pairs with the real video, covering V2V, I2V, and T2V variants, complementing the 30K human-annotated pairs. The advantage of this path is zero annotation cost and controllable negative samples; the disadvantage is that the distortion type may not match the model's actual failure modes.

【Path 2: model self-sampling to construct candidate groups (online/offline preference pairs)】The anchor paper's GRPO is the online form of this path: given prompt c, N trajectories are sampled from the reference policy to form a group, and within-group normalized advantage A_i = (R_i − mean) / std is used for relative comparison, requiring no pre-labeled preference pairs. Cosmos-Predict 2.5 also takes the online form (8 rollouts × 20 diffusion steps per condition, within-group normalization, 256 training steps, batch 32). The offline form includes Step-Video-T2V (multiple videos generated per prompt with different random seeds, then human-annotated), Kling 3.0 Omni (multiple variants under the same MVL condition + human comparison), and JavisDiT++ (3 generated candidates + 1 ground truth per prompt, 4 candidates total).

【Synthetic data in the anchor paper's reward-model training】Stage 2 explicitly "leverages unlabeled data produced by models of different capability levels and different RL iterations" to train the reward model — i.e., using intermediate byproducts of model evolution as additional RM training material, an extremely low-cost, well-distributed source of synthetic data, and of the same kind as Seedance 1.0's "preference data covers synthetic videos generated at different stages of the model."

【Self-generated data on the distillation side】In the anchor paper's Phase 4 Self-Forcing distillation, each frame is autoregressively rolled out (with KV cache) based on the model's own previously self-generated output, and the DMD loss is applied at the video level — the training data is the student model's own rollout, purely self-generated data. StreamChar's two-stage distillation Stage II (400 steps, student lr 2e-6, fake score network lr 4e-7) likewise uses online chunk rollouts.

### Degree of Human Involvement (human annotation, human QC, model pre-screening + human review) ⚠️

【Anchor paper】Human involvement appears in only two places: ① "human preference annotation (y1,y2)" used for reward-model training — the annotation volume, protocol, and annotator-team size are all undisclosed [uncertain]; ② the final evaluation adopts the GSB (Good–Same–Bad) protocol, explicitly stating "we ask the human artist to give an overall comparison for the results" — a human artist gives an overall comparison. The paper's stated rationale for choosing GSB over a forced binary choice is clear: GSB allows annotators to express "no difference" when the gap is subtle, reducing noisy judgments forced at the margin — something especially important for video evaluation.

【Cross-model lineage of human-involvement forms (post-training is the stage with the highest human density)】
· Pure human preference annotation: Step-Video-T2V (annotators score preferences among multi-seed generation results, supervised throughout by quality-control staff for consistency), Kling 3.0 Omni (human evaluators compare multiple variants), HunyuanVideo 1.5 (GSB annotation; on the I2V side, accompanying images additionally require manual verification of text-image consistency);
· Multi-dimensional structured annotation protocol (the most fine-grained): Seedance 1.0 — "select the best/worst along a specified dimension, while ensuring the best sample is not inferior to the worst along other dimensions." This constraint directly resolves the problem of contradictory pairings under multi-dimensional rewards, and is one of two solutions to the same problem alongside JavisDiT++'s "normalized modality-aware ranking" (whose explicit purpose is "to ensure consistency within each modality, rather than pairing high-quality audio with low-quality video");
· Semi-automatic production line: SkyReels-V2 (30K human-annotated pairs + automatically generated controlled-distortion samples);
· Human final review of SFT sets: HunyuanVideo's 1 million human-curated samples (seven-dimension rubric), SkyReels-V4's 1 million human-curated samples, Movie Gen's human selection for cinematic quality + human caption rewriting, Step-Video-T2V's item-by-item human review, Apollo Stage III's manually curated high-quality set;
· Professional annotation for public preference sets: VideoReward employs "professional annotators" to record pairwise preferences (A wins/tie/B wins) separately along the VQ/MQ/TA dimensions for each triplet, modeled with a Bradley-Terry model with ties (BTT);
· Human involvement used only for evaluation, not fed back into training (the most common waste in this topic): Movie Gen, CogVideoX, Open-Sora 2.0 (100-prompt human win rate), Ovi (50-person blind test), Unison (40 samples × 25 people = 1000 ranking votes), UniTalking (20 people × 50 prompts), InstructAV2AV (25 people × 20 samples × 3 dimensions), Foley-Omni (A/S/T-MOS), HunyuanVideo-Foley (MOS-Q/S/T), CineDance (10 evaluators per video, 5-point scale), UltraVideo (10-person comparison), LVD-2M (200 responses), Script-a-Video (20 professional raters) — all of these works have already organized human labor for subjective evaluation, and the marginal cost of extending the same batch of annotation into preference pairs is extremely low, yet none of them do so. This is a systemic gap on the open-source/academic side of post-training.

## Audio-Video Alignment

### Audio-Video Synchrony Detection Method (lip sync, event alignment) ⚠️

The anchor paper does not touch on audio-video synchrony detection on the post-training data side at all [uncertain] — none of the four reward models (video aesthetics, image aesthetics, motion quality, text-video alignment) involve synchrony, nor do PE's three rewards. AV models are handled only at the Phase 4 distillation stage, following OmniForcing's equipping with asymmetric block-causal alignment and an audio sink token — architectural alignment rather than data-side synchrony detection. This means: even applying this framework's full four-phase post-training to an AV model, lip-sync and event-alignment quality would not be directly optimized by any reward signal.

【The only complete synchrony-reward practice on the cross-model post-training side】JavisDiT++'s AV-DPO: among its six reward models, Synchformer is dedicated to temporal synchrony and ImageBind is dedicated to cross-modal semantic similarity, and both participate in preference ranking side by side — the only public case treating AV synchrony as an independent preference dimension. About 25K preference pairs, a 30K-prompt pool (not overlapping with SFT data), 3 generated candidates + 1 ground truth per prompt (4 candidates total), 1 epoch, LoRA with 121M trainable parameters.

【The rest】Seedance 1.5 pro's RLHF states it covers "audio fidelity," but whether it includes a synchrony dimension is undisclosed [uncertain]; whether Kling 3.0 Omni's DPO treats lip sync as an independent scoring item is likewise undisclosed [uncertain]. Data-side AV synchrony filtering (SyncNet/Synchformer threshold screening) uniformly occurs in the pretraining data pipeline rather than at the post-training stage across all models.

### Specific Synchrony Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values — e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

The anchor paper gives no synchrony metrics or thresholds whatsoever [uncertain]. The only nameable metric at the post-training stage across the cross-model objects is JavisDiT++'s use of Synchformer as a temporal-synchrony reward model, but when used as a reward model it is a continuous score participating in normalized ranking, not a threshold — a fundamentally different paradigm from the threshold-based approach at the data-filtering stage (e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 and conf>1.5, UniTalking's SyncNet conf>0.9):
· Data-filtering stage: hard binary thresholding (pass/reject), aimed at ensuring the training material itself is in sync;
· Post-training reward stage: continuous scores participate in within-group relative comparison, aimed at pushing the model's output toward greater synchrony.

This "threshold → continuous reward" conversion is the key shift as synchrony moves from the data side to the training-signal side, but currently only JavisDiT++ implements it in full. Unison's evaluation suite includes LSE-C/LSE-D, and UniVerse-1's Verse-Bench includes LSE-C/AV-A, but both are used only for evaluation and are not fed back into training. [uncertain]

### Separate Handling of Temporal vs. Semantic Synchrony (time alignment and content-semantic matching treated as two independent filtering conditions) ⚠️

Not addressed by the anchor paper [uncertain]. JavisDiT++'s six-reward design effectively implements this separation: temporal synchrony is handled independently by Synchformer, while semantic matching is handled independently by ImageBind across three chains — text-audio, text-video, and audio-video cross-modal — i.e., time alignment and content-semantic matching are two independent reward terms rather than being merged into a single "AV quality score." Its "normalized modality-aware ranking" further ensures that pairing does not mismatch high-quality audio with low-quality video, equivalent to maintaining within-modality consistency at the preference-construction level. This design is fully aligned in objective with the constraint in Seedance 1.0's human annotation protocol (the best sample must not be inferior to the worst along other dimensions): both aim to prevent self-contradictory preference signals under multi-dimensional rewards. This is one of the two most transferable pieces of engineering experience in this topic regarding post-training data construction.

### Audio Quality Filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, off-screen-source removal, background-music separation) ⚠️

Not addressed by the anchor paper [uncertain]. Audio quality control at the post-training stage mainly takes the form of reward models rather than threshold filtering: JavisDiT++ uses AudioBox-Aesthetics to assess audio quality; Seedance 1.5 pro's multi-dimensional reward includes audio fidelity. The data-side exception is Movie Gen's audio SFT set — its cinematic split is auto-filtered by a cinematic-quality classifier + AED sound-event detection and then manually selected, explicitly excluding clips containing human vocals; Foley-Omni Stage 3's admission criterion is passing a complete cleaning pipeline (six filtering thresholds + Gemini annotation + Bandit energy verification) and containing multiple audio components. Specific thresholds for SNR, silence ratio, no-audio-track removal, and background-music separation do not appear in any post-training material [uncertain] — these belong to the pretraining data pipeline's responsibility.

### Classification and Separate Handling of Speech/Sound Effects/Music ⚠️

Not addressed by the anchor paper [uncertain]. Cross-model: Movie Gen is the only work that explicitly treats audio types separately at the post-training stage — the cinematic split (containing in-scene diegetic audio and off-screen ambient/thematic music, excluding vocals) and the high-quality-audio split (pure music, O(10)K hours + pure sound effects, O(10)K hours) are built separately, differing in scale by two orders of magnitude, indicating that "learning soundtrack/sound-effects" and "learning video-audio correspondence" are treated as two separable objectives. Foley-Omni's three-stage curriculum (sound effects → music → full soundtrack) is likewise a type-separated curriculum form, but stops at supervised training. No work performs separate reward design for speech/sound-effects/music at the preference-learning level [uncertain] — this is the clearest unfilled gap in AV post-training: the existing audio reward model (AudioBox-Aesthetics) is a general aesthetic score that cannot distinguish between "insufficient dialogue clarity" and "sound effects misaligned with on-screen events" — two entirely different failure modes.

## Training Coordination

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

【The anchor paper's four-stage curriculum is itself its core contribution】Phase 1 SFT → Phase 2 RLHF(GRPO) → Phase 3 PE → Phase 4 AD. The dependency between the four phases is argued fairly clearly: SFT provides a stable reference policy for RLHF and enlarges the exploration space; PE shares the same reward loop with RLHF but acts on the input side, making the two complementary; AD transfers the capability from the first three phases into a causal architecture. AD is further subdivided into three sub-phases: ① DMD distillation — first distills the original pretrained model into a bidirectional student model requiring only a few denoising steps while retaining a global attention receptive field, providing a high-quality, easily regressed teacher trajectory for the subsequent transfer to a causal architecture; ② Causal ODE regression — directly training a causal student with the DMD loss is unstable due to architectural differences, so an efficient initialization strategy is introduced along with a block-causal mask, training the model to perform effective denoising prediction using only causal history; ③ Self-Forcing distillation — each frame is generated via autoregressive rollout (with KV cache) based on the model's own previously self-generated output, thereby applying the DMD loss at the video level to directly assess the quality of the entire sequence.

【GRPO's specific curriculum design (the anchor paper's technical highlights)】Following DanceGRPO, flow-matching sampling is formulated as an MDP, with sparse reward given only at the terminal step. For video generation specifically, the authors note that MixGRPO suffers reward collapse when the random subset is small, so, inspired by Flash-GRPO, they adopt "isotemporal grouping" — each prompt is assigned a distinct timestep t_i, and during denoising, each prompt group performs an ODE→SDE conversion only at its assigned timestep (that step uses SDE sampling to support exploration and gradient computation), while all other timesteps use deterministic ODE updates to produce higher-quality generations and more reliable reward signals. They also introduce "Temporal Gradient Rectification," which explicitly normalizes the time-dependent scaling factor λ(t)=√Δt/σ_t + σ_t√Δt(1−t)/(2t). Following DanceGRPO, the KL regularization term is omitted.

【Cross-model curriculum comparison】HunyuanVideo 1.5's CT→SFT→RLHF three stages, with T2V/I2V kept entirely separate throughout; SkyReels-V2's 540p concept-balancing SFT → 720p high-quality SFT → three-stage DPO; Cosmos-Predict 2.5's five-domain parallel SFT → model merging → GRPO → rCM distillation; Motif-Video 2B inserts SFT into the middle of the resolution ladder (720p pretraining starts from the 480p SFT checkpoint); Seedance 1.0's multi-sub-model SFT → model merging → RLHF; LongCat Avatar 1.5 extends GRPO from video-level reward to frame-level reward, with multi-segment rollout supporting up to 5 clips where only the final segment participates in optimization, distilled down to 8 steps with DMD2.

### Data Mixture Changes Across Training Stages (pretraining / annealing / SFT high-quality subset) ⚠️

【Anchor paper】The data mixture at every stage is entirely undisclosed [uncertain]. The confirmable structural fact is: SFT uses "paired text-video data," while the four downstream stages — RLHF/DMD/ODE regression/self-forcing — all share a single "curated prompt set" (requiring only prompts, no paired video). This is a key divide in post-training data structure: SFT needs paired data, while RL and distillation need only prompts. This substantially lowers the data-acquisition cost of the latter three stages, and also explains why constructing the prompt set at the post-training stage (diversity, category balance, deduplication, non-overlap with the SFT set) receives more attention than material collection.

【Cross-model stage-mixture figures】
· ALIVE is the most complete: Continue-Training 4.3M balanced samples (3 epochs) → SFT 5M (a 3:1 mix of high-aesthetic and realistic data, only 0.5 epoch, asymmetric learning rates video 1e-5 / audio 1e-6) → 1080p Refiner 0.7M (1 epoch) → character-driven 0.8M. SFT is deliberately trained for only 0.5 epoch to avoid overfitting to the high-aesthetic distribution at the cost of diversity — an important trick beyond the mixture itself;
· SkyReels-V4: 5 million (3 epochs) → 1 million human-curated (3 epochs);
· Cosmos-Predict 2.5: each of the five domains gets 30k iterations at batch 256, plus 388K cooldown samples at 4K;
· Movie Gen: performs model averaging after SFT; Seedance 1.0: model merging after multi-sub-model SFT — weight averaging is an alternative implementation of "data mixture";
· Foley-Omni: Stage 3 uses 216 hours of target-domain data + 100 hours of replay per domain, a rare case of explicit experience replay at the post-training stage;
· HunyuanVideo 1.5: CT 1 million per task → a stricter SFT subset → RLHF with O(10K) prompts; the super-resolution module is separately trained on 1 million 1K–4K clips.

【Regularity】SFT is generally trained "shallowly" (low epoch count, small learning rate, early stopping) — Seedance 1.0 uses a smaller learning rate than pretraining, a limited GPU count, and early stopping to prevent overfitting and preserve text controllability; ALIVE trains only 0.5 epoch; CogVideoX trains only 10k steps. This stands in sharp contrast to pretraining's "train to convergence," essentially balancing "fitting the high-quality distribution" against "retaining the diversity and controllability of pretraining."

### Post-Training Data (SFT curated-set scale and filtering criteria, preference-pair count and annotation method, reward-model training data)

This entry is centered on precisely this field; a directly reusable summary of conclusions follows.

【I. Scale and filtering criteria of the SFT curated set】Scale on the order of 10^6–10^7 samples, at 0.4%–20% of the pretraining pool (NAVA 1.07%, Allegro 0.4%, CogVideoX 20%, Goku 12.5%, Cosmos ~10%, SkyReels-V4 second stage 20%). The common filtering structure is four layers stacked: ① a strictified version of pretraining thresholds (aesthetic score, sharpness, motion score, text-area ratio, CLIP similarity — Allegro is the only work to give a single, fully reproducible combination of thresholds); ② concept/domain balancing (LongCat's inverse-density sampling in caption-embedding space, Movie Gen's k-NN concept balancing, Cosmos's classifier-based five-domain split, Seedance's targeted collection across hundreds of categories, Motif's style/subject-tag-driven domain-balancing); ③ upgraded re-captioning (NAVA re-captions with Gemini-3-Pro, Open-Sora 2.0 re-captions with Qwen2.5-Max, Movie Gen manually rewrites, Step-Video manually refines); ④ human final review (HunyuanVideo's seven-dimension rubric of a four-item aesthetics set + a three-item motion set, Step-Video's four-item set of sharpness/aesthetics/motion appropriateness/smooth transitions). Training configuration is generally "shallow" to prevent overfitting.

【II. Preference-pair count and annotation method】Scale on the order of 10^4–10^5 pairs, 2–3 orders of magnitude smaller than the SFT set. Four construction methods:
① Human-annotated real preferences (most expensive, most reliable): Step-Video-T2V (multi-seed generation + annotator scoring + QC-supervised consistency), Kling 3.0 Omni (human comparison of multiple variants under the same MVL), HunyuanVideo 1.5 (GSB annotation + non-repeating pairing), SkyReels-V2 (30K human-annotated pairs);
② Automatically constructed negative samples: SkyReels-V2's controlled-distortion corrupted samples (covering V2V/I2V/T2V);
③ Automated ranking by multiple reward models: JavisDiT++ (six RMs + normalized modality-aware ranking, 25K pairs);
④ Online reward replacing offline preference pairs (currently mainstream): the GRPO route of the anchor paper, LongCat-Video (group size 4, 64 prompts per step, roughly 0.5k iterations), Cosmos-Predict 2.5 (8 rollouts × 20 steps, within-group normalization, 256 steps, batch 32) — no pre-labeled preference-pair dataset exists; preference is given online by the RM at rollout time.

Two key engineering lessons on annotation protocol: Seedance 1.0's "select the best/worst along a specified dimension, with the best not inferior to the worst along other dimensions," and JavisDiT++'s "normalized modality-aware ranking" to avoid pairing high-quality audio with low-quality video. On the evaluation-protocol side, GSB (Good–Same–Bad) has become the de facto industrial standard (used by the anchor paper, Kling, Seedance, and HunyuanVideo 1.5), with the advantage that it lets annotators express "no difference," reducing noise at the margin.

【III. Reward-model training data】Two public assets: HPDv3 (1.08M text-image pairs, 1.17M pairwise comparisons) and the VideoReward preference set (16K prompts, 108K videos generated by 12 T2V models, 182K triplets, with professional annotators separately recording A-wins/tie/B-wins along the VQ/MQ/TA dimensions, modeled with a Bradley-Terry model with ties (BTT), plus a held-out validation set of 13K non-overlapping triplets whose prompts). The anchor paper's RM training paradigm: a Qwen3.5 VLM backbone + an MLP head, an uncertainty-aware ranking loss, two-stage training (Stage 1 uses data-aware orthogonal gradient projection to fuse the diverse aesthetic preferences of HPDv3++ while preserving HPSv3's original knowledge; Stage 2 leverages unlabeled data produced by models of different capability levels and different RL iterations). Multi-reward fusion is explicitly identified by the paper as a key systemic difficulty, requiring careful design of aggregation strategy and weighting coefficients.

【IV. Industry-stratification conclusion】Closed-source industrial models (Seedance, Kling, HunyuanVideo 1.5, Step-Video, SkyReels, LongCat) have generally completed the two-stage SFT + preference-alignment post-training; on the open-source and academic side (Movie Gen, CogVideoX, Allegro, Goku, Motif, MAGI-1, Open-Sora, and nearly all academic AV work), the vast majority stop at SFT, with preference alignment the biggest gap — and within the JAVG (joint audio-video generation) field specifically, aside from JavisDiT++, the field is entirely blank.

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The anchor paper discloses no infrastructure or throughput data whatsoever (no GPU count, no training duration, no cost) [uncertain], but the main text twice emphasizes that compute constraints are the core tension of post-training: "post-training must operate under strict constraints on sampling cost, evaluation quality, and system efficiency" and "rollout generation is expensive," citing DanceGRPO as support. Its countermeasures are threefold: ① isotemporal grouping — each prompt performs SDE sampling and gradient computation at only one assigned timestep, with all other steps using ODE, compressing GRPO's gradient-computation cost to near single-step; ② freezing the generation backbone and training only PE's LLM policy, so that PE can be applied to any off-the-shelf generator without retraining the video model; ③ autoregressive distillation compresses the bidirectional model into a causal, few-step model to lower deployment inference cost.

【Cross-model post-training compute figures】Movie Gen's video SFT uses only 512 H100s (extremely small relative to its pretraining scale); Allegro SFT consumes 2.6M samples, batch 256, 10K steps, 256× H100; Cosmos-Predict 2.5's GRPO, due to memory constraints, decomposes trajectory probability into a sum of step-wise conditional probabilities, computing a gradient every two steps and accumulating along the full 10-step trajectory before a single parameter update, training for 256 steps at batch 32; Seedance 1.5 pro states that infrastructure optimization targeting the RLHF pipeline yields nearly a 3× training speedup; BranchGRPO organizes rollouts into a branching tree, sharing prefixes to reduce overhead and pruning low-reward paths; MixGRPO uses mixed ODE-SDE to improve training efficiency. On the data-processing tooling side: neither NeMo Curator nor Data-Juicer provides preference-pair construction capability for video generation (Data-Juicer 2.0 claims support for foundation-model post-training, but its video-side coverage is limited to the supervised-distillation route) — post-training data construction currently relies entirely on in-house tooling built by each team [uncertain].

## Results Comparison

### Quantitative Impact of Data-Strategy Ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and the corresponding evaluation metrics) ⚠️

【The anchor paper's quantitative results (only two numbers, no component-by-component ablation)】On its internal video generation model, the RLHF stage achieves a 31% improvement in the overall GSB metric, with the gain "most pronounced along the visual quality and motion quality dimensions, both showing substantial enhancement," while the improvement in text alignment is comparatively modest — the authors attribute this to the limited accuracy of the current text-alignment reward model, which constrains the room for optimizing semantic correctness. On top of this, adding the prompt enhancer brings a further 20% overall GSB improvement, again driven by visual and motion quality, while text alignment does not regress. The paper also provides an RLHF-effect visualization on Wan-2.1 (Figure 2).

【Methodological shortcomings (must be flagged)】The paper does not provide: the baseline model's name, the denominator definition of GSB (whether 31% is a Good rate or (G−B)/N), the annotated sample size, the number of annotators, confidence intervals, per-dimension scores, comparison against other post-training approaches, the gain from SFT alone, or the quality-speed tradeoff curve before/after AD distillation. There is no data-side ablation at all (no filtering-strictness ablation, no caption-density ablation, no data-mixture ablation), and no results are reported on public benchmarks such as VBench. Its "31%/20%" figures are therefore not comparable across works [uncertain].

【Comparable cross-model ablation evidence】
· Cosmos-Predict 2.5 provides the most solid per-domain SFT ablation in this topic (human win rate, SFT vs. pretraining baseline): robotic manipulation 72.6% vs. 8.3%, object permanence 50.9% vs. 27.7%, high motion 44.0% vs. 34.7%, complex scenes 42.6% vs. 35.4%, driving 47.9% vs. 28.8% — directly quantifying the benefit of "curating SFT data by domain," with the benefit not proportional to in-domain data volume (the 730K robotic-manipulation domain shows the largest gain, while the 10.4M object-permanence domain shows a moderate gain), indicating that the gap between a domain and the base model's capability matters more for SFT gain than data volume;
· Cosmos's rCM distillation ablation: the 4-step generation's PAI-Bench score is nearly on par with the teacher (T2W 0.764 vs. 0.768, I2W 0.816 vs. 0.810, with the distilled I2W even slightly higher);
· AVSCap's key thesis: RL gains > expanding SFT data volume — direct evidence for "post-training investment takes priority over annotation-volume expansion";
· MOVA's prompt-rewriter ablation: human Arena ELO rises from 982.9 to 1025.3, independently quantifying PE's value (consistent in direction with the anchor paper's qualitative conclusion that PE brings a 20% GSB gain);
· StreamChar's ablation: single-stage distillation underperforms two-stage distillation (corroborating the anchor paper's three-sub-phase AD design);
· video-SALMONN 2's MrDPO: the 7B model's caption error rate drops 28% relative to baseline;
· LongCat-Video's HPSv3-percentile design (taking the top 30% highest-scoring frames) and the Motion RM's grayscale-video-input design are both targeted ablation products aimed at reward-signal contamination, but the paper gives no control-group numbers [uncertain].

### Evidence on Quality vs. Quantity (cases where small-and-refined data beats large-and-messy data) ⚠️

This topic is the most concentrated body of evidence in the video-generation field for the "quality over quantity" thesis:

【Strongest quantitative evidence】
· NAVA: from a 15M training corpus, multi-operator collaborative filtering curates 160K (1.07%) for SFT, and this subset's captions are regenerated by the more expensive Gemini-3-Pro — "one-in-a-thousand curation + upgraded annotation" is the most aggressive quality-first practice in this survey;
· Allegro: 500M → 2M (0.4%);
· Movie Gen: video SFT training uses only 512 H100s, an extremely small investment relative to its pretraining scale, yet it carries the final definition of aesthetics and cinematic quality;
· Cosmos-Predict 2.5: SFT on 730K robotic-manipulation samples yields a crushing 72.6% vs. 8.3% human win rate — direct evidence that "a small in-domain dataset can beat a large, messy pretraining pool";
· AVSCap's thesis that "RL gains > expanding SFT data volume" advances the thesis further, from "small-and-refined data" to "optimization paradigm > data scale";
· ALIVE's SFT trains for only 0.5 epoch — a converse footnote to "quality-first data should not be over-fit either": quality-first does not mean letting the model fully fit the curated distribution.

【The anchor paper's position】Its core thesis, "SFT as the foundation for RLHF," is actually a more advanced version of the quality-vs-quantity thesis: SFT's value lies not in directly improving subjective quality (the paper explicitly states "SFT is not intended to fully solve alignment or optimize subjective quality"), but in providing a stable reference policy that "enlarges the exploration for RLHF." In other words, the ultimate value of SFT-curated data is only realized through RLHF — repositioning "small-and-refined data" from an endpoint to middleware. This is the most noteworthy conceptual shift to record in this topic, even though the paper provides no ablation evidence for it [uncertain].

【Counter-evidence / cautionary note】Apollo explicitly uses manually curated high-quality data for Stage III quality refinement but performs no preference alignment at all — its alignment capability comes entirely from data quality and architecture; Step-Video-T2V notes that DPO's gains saturate once the model can easily distinguish positive from negative samples — together these show that the return curves of "data quality" and "preference optimization" have different shapes: the former hits diminishing returns earlier, while the latter requires a continuously updated reward signal to sustain its gradient.

### Alignment Between Training-Data Domain Distribution and Benchmark Category Taxonomy (e.g., VABench's seven major categories) ⚠️

【Anchor paper】Its evaluation taxonomy is three-dimensional: visual quality (overall appearance, sharpness, absence of artifacts), motion quality (temporal coherence, smoothness, plausibility of motion patterns), and text alignment (semantic consistency between the generated video and the input prompt) — strictly one-to-one with three of its four reward models (video aesthetics/image aesthetics merged into visual quality, motion quality, text-video alignment). This "reward dimensions = evaluation dimensions" design is standard practice in post-training work; its advantage is that the training objective is perfectly consistent with the evaluation methodology, but its drawback is an inherent risk of self-validation — a model optimized with an RM and then verified by human evaluation using the same dimensions cannot rule out reward hacking that simply is not captured by this set of dimensions. The paper reports no results on external benchmarks such as VBench or EvalCrafter (despite citing them and criticizing that their "evaluation signals are often noisy") [uncertain].

【Industry convergence on the three-dimensional taxonomy】VQ/MQ/TA has become the de facto standard taxonomy for video-generation post-training: VideoReward (annotating VQ/MQ/TA as three separate dimensions), LongCat-Video (three VQ/MQ/TA reward models), the anchor paper (three-dimensional GSB), HunyuanVideo 1.5 (aesthetic appeal / sharpness / motion fluency), Seedance 1.0 (Foundational/Motion/Aesthetic three RMs) — this three-dimensional system mirrors the training-data SFT filtering criteria (aesthetics + motion + text-image consistency), i.e., the filtering dimensions, reward dimensions, and evaluation dimensions have all become highly aligned.

【Absence of the AV dimension】AV evaluation systems such as VABench's seven categories have not yet been connected to post-training reward dimensions: the three-dimensional system is entirely visual, with audio fidelity, lip sync, event alignment, and timbre consistency absent from it. JavisDiT++'s six rewards (audio quality / text-audio / video quality / text-video / cross-modal similarity / temporal synchrony) are the only post-training system to extend the evaluation taxonomy to the full AV dimension, and can be seen as a candidate successor to the three-dimensional system in the AV era. AV evaluation categories such as Movie Gen's audio-side MOS-Q/S/T, Unison's 11 objective metrics (VA/ID/PQ/CU/WER/TA/TV/AV/LSE-C/LSE-D/DS), and UniVerse-1's Verse-Bench (MS/AS/ID/FD/KL/CLAP/LSE-C/AV-A/WER) have not been adopted as reward dimensions by any post-training work [uncertain].

## Other Information

### summary_note

【Core conclusions】① The anchor paper, arXiv:2604.25427, is a four-phase post-training blueprint (SFT → GRPO RLHF → Prompt Enhancement → Autoregressive Distillation) from HKU + JD Explore Academy and others. Its technical contributions concentrate on the GRPO side (isotemporal grouping to mitigate MixGRPO's reward collapse, Temporal Gradient Rectification, omission of the KL term, four-reward fusion) and the distillation side (DMD → Causal ODE regression → Self-Forcing, three sub-phases), but discloses almost nothing on the data dimension — the SFT data description is a single sentence, "we constructed a high-quality text-video dataset for SFT," with no scale, no filtering criteria, and no prompt-set count; and Section 3.1's listed SFT failure modes (refusal cascades, incoherent reasoning) are clearly migrated from LLM post-training text and contain nothing video-specific. Its value as an information source on "post-training data strategy" is therefore limited; its real value lies in providing the organizing logic of the four-phase framework and the thesis that "SFT is the foundation of RLHF, not the endpoint."

② The truly reusable data methodology comes from the cross-model objects: the scale range of SFT curated sets (0.4%–20% of the pretraining pool, absolute scale 10^6–10^7), the four-layer filtering structure (strictified thresholds + concept balancing + caption re-annotation + human final review), Allegro's complete threshold combination, HunyuanVideo's seven-dimension human rubric, LongCat's inverse-density sampling, Cosmos's classifier-based domain split and per-domain win-rate ablation, Movie Gen's four-stage pipeline and dual audio splits, and Seedance's targeted collection across hundreds of categories + sub-model fusion.

③ A key judgment on the preference-data side: the industry is rapidly shifting from "offline preference pairs + DPO" to "online reward + GRPO," the latter requiring no pre-labeled preference-pair dataset (already the case for the anchor paper, LongCat, and Cosmos), so the metric of "preference-pair count" is itself losing meaning — the competitive dimension shifting instead to reward-model quality and multi-reward fusion strategy. The part still requiring human annotation has shifted from "annotating preference pairs" to "annotating reward-model training data." The two most transferable annotation-engineering lessons are Seedance 1.0's multi-dimensional consistency constraint and JavisDiT++'s normalized modality-aware ranking, both of which solve the same problem: how to avoid self-contradictory preference signals under multi-dimensional rewards.

④ The biggest gap: the audio-video dimension. The anchor paper's four reward models are entirely visual; AV is reflected only in the distillation architecture (OmniForcing's audio sink token). Industry-wide, apart from JavisDiT++'s AV-DPO (25K pairs, Synchformer for synchrony, AudioBox for audio quality, ImageBind for semantics), no work incorporates lip sync, sound-effect event alignment, or timbre consistency into preference rewards. Meanwhile, a large number of academic works have already organized human subjective evaluations (Unison's 1000 ranking votes, Ovi's 50-person blind test, InstructAV2AV's 25 people × 20 samples × 3 dimensions, etc.) yet none feed them back as a training signal — an extremely low-marginal-cost action that remains widely undone.

⑤ Suggested reuse path: for the SFT stage, follow Allegro's thresholds + HunyuanVideo's seven-dimension rubric + LongCat's inverse-density sampling; for reward models, directly reuse the open-source VideoAlign/VideoReward and HPSv3 (fine-tuned on internal data per LongCat's approach, and decoupling motion from aesthetics via its grayscale-input trick); for the RL stage, use GRPO online rewards to avoid preference-pair annotation cost, and regularize with a diffusion loss per Cosmos's approach to suppress reward hacking; AV models need to independently fill in Synchformer/AudioBox-class synchrony and audio-quality rewards.

## Uncertain Fields

The research information for the following fields is partly uncertain (sourced from ⚠️ markers):

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
- safety_filtering
- caption_model
- caption_structure
- joint_av_caption_schema
- dialogue_transcription_attributes
- geometric_structured_annotation
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
