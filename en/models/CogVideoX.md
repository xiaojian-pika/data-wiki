# CogVideoX (including CogVideoX-2B / 5B, CogVideoX1.5-5B / 5B-I2V, and the accompanying CogVLM2-Caption annotation model; on the product side corresponds to "Qingying / Ying," on the sound-effects side corresponds to CogSound)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

CogVideoX (including CogVideoX-2B / 5B, CogVideoX1.5-5B / 5B-I2V, and the accompanying CogVLM2-Caption annotation model; on the product side corresponds to "Qingying / Ying," on the sound-effects side corresponds to CogSound)

### Releasing Organization/Company

A joint team from Zhipu AI and Tsinghua University's THUDM, with paper corresponding author Jie Tang; core contributors include Zhuoyi Yang, Jiayan Teng, Wendi Zheng, Ming Ding, Shiyu Huang, Xiaotao Gu, and others

### Release Date (technical report/paper/open-source date)

CogVideoX-2B weights and code open-sourced on August 6, 2024; arXiv preprint 2408.06072 v1 ("CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer") on August 12, 2024; CogVideoX-5B open-sourced on August 27, 2024; CogVideoX-5B-I2V open-sourced in September 2024; "New Qingying" released and CogVideoX1.5-5B / 1.5-5B-I2V open-sourced on November 8, 2024 (the sound-effects model CogSound was released concurrently); the paper was accepted by ICLR 2025, with the v3 revision updated on March 26, 2025

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a series of text-to-video/image-to-video diffusion Transformer foundation models) + accompanying toolchain (3D causal VAE, CogVLM2-Caption video annotation model, caption upsampler prompt rewriting, fine-tuning and inference codebase zai-org/CogVideo)

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced)

The degree of openness is relatively high among closed-source major vendors of the same period, but the data itself is not open-sourced:
· Weights: open-sourced. CogVideoX-2B (Apache 2.0), CogVideoX-5B / 5B-I2V, and CogVideoX1.5-5B / 1.5-5B-I2V are all publicly released on Hugging Face (THUDM/zai-org organization) and in SAT format, permitting commercial use.
· Code: open-sourced. Inference, fine-tuning (LoRA/SFT), and both the SAT and diffusers implementations are on GitHub at https://github.com/zai-org/CogVideo (formerly THUDM/CogVideo).
· Data processing pipeline: publicly described in the paper (Section 3.4 Data + Appendix G "Dense Video Caption Data Generation" + Appendix K "Data Filtering Details"), disclosing the negative-label taxonomy, the six Video-LLaMA classifiers with their test-set accuracy tables, the caption generation chain, and the full text of the GPT-4 summarization prompt; however, the cleaning code and threshold configuration files are not open-sourced, and the dataset itself is not released.
· Annotation model: open-sourced. CogVLM2-Caption (huggingface.co/zai-org/cogvlm2-llama3-caption) and the 3D VAE weights have both been released, which is the most substantive manifestation of "the data pipeline being public" — external parties can directly reproduce its annotation stage.
· Training data: not open-sourced; the specific sources and manifests for the 35M clips and 2B images have not been disclosed.
· CogSound (sound-effects model): closed-source, provided only within the Qingying product and API, with no technical report.

### Whether Simultaneous Audio-Video Generation Is Supported, and the Implementation Approach (native joint/cascaded/MoE fusion)

The CogVideoX model itself does not support simultaneous audio-video generation; it is a purely visual text-to-video/image-to-video model, and the training data does not use an audio track.
At the product layer ("New Qingying," November 2024), videos "with built-in sound effects" are achieved via a cascade approach: CogVideoX (version 1.5) first generates a 10-second, 4K/60fps silent video, and then the independent sound-effects model CogSound performs video-conditioned V2A (video-to-audio) dubbing. The publicly disclosed technical points of CogSound are: semantic/emotional extraction based on GLM-4V's video understanding capability → audio generation via a latent diffusion model → establishing correspondence between frame-level video features and audio features using "Block-wise Temporal Alignment Cross-attention" → overlaying rotary position encoding (RoPE) to improve temporal consistency for long sequences. It can generate complex sound effects such as explosions, flowing water, musical instruments, animal calls, vehicles, and rhythmic/musical elements.
The implementation approach is therefore "cascaded," rather than native joint generation or MoE fusion; and the data pipelines of the two models are entirely independent — the video-side paper does not touch the audio dimension at all.

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with the nature of each source labeled: official primary/same-team corroborating/third-party reporting)

- Official primary: CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer (arXiv:2408.06072v3, ICLR 2025, 30 pages including Appendices G/J/K) https://arxiv.org/abs/2408.06072 and https://arxiv.org/pdf/2408.06072
- Official primary: zai-org/CogVideo open-source repository (code, weights, caption toolchain) https://github.com/zai-org/CogVideo
- Official primary: CogVLM2-Caption annotation model weights (the video caption model used during CogVideoX training) https://huggingface.co/zai-org/cogvlm2-llama3-caption
- Same-team corroborating: Zhipu's "New Qingying" CogVideoX+CogSound technical deep-dive (reposted by the BAAI community from Zhipu's official technical explanation, including a description of CogSound's Block-wise Temporal Alignment Cross-attention and data-selection framework) https://hub.baai.ac.cn/view/40956
- Third-party reporting: Zhipu's video generation foundation model Qingying upgrade (New Qingying, 10s/4K/60fps/built-in sound effects, 2024-11-08) https://cn.technode.com/post/2024-11-08/zhipu-qingying-new/
- Third-party reporting: Introduction to CogSound sound-effects model capabilities https://ai-bot.cn/cogsound/
- Third-party compilation: CogVideoX paper literature review (Moonlight, including data filtering and the 35M clips description) https://www.themoonlight.io/en/review/cogvideox-text-to-video-diffusion-models-with-an-expert-transformer

## Data Scale and Distribution

### Training Data Volume (video count/hours/token count, pretraining and SFT reported separately) ⚠️

· Video pretraining: after filtering, approximately 35 million (35M) single-shot clips remain, each averaging about 6 seconds (original paper text: "approximately 35M single-shot clips remain, with each clip averaging about 6 seconds"), from which the total duration is estimated at roughly 58,000 hours.
· Auxiliary image data: an additional 2 billion (2B) images are used, drawn from LAION-5B and COYO-700M, filtered by aesthetic score (the hyperparameter table gives a lowest aesthetic value of 4.5). During training, images are treated as single-frame videos and mixed with video training.
· High-quality fine-tuning (stage 4, FT): a higher-quality subset selected from the above data, accounting for 20% of the total data volume, trained for 10k steps.
· Annotation model training data: to fine-tune a substitute model (LLaMA2) for GPT-4 summarization, 50,000 summary data points were collected.
· Filter training data: 20,000 videos were manually annotated with positive/negative labels, of which 10% were randomly split off as a test set.
· CogSound's data scale is undisclosed [uncertain]. The data increment of CogVideoX1.5 relative to 1.0 is also undisclosed [uncertain].

### Data Source Composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

The paper only vaguely states that the data comes from videos on the internet ("Videos from the Internet usually include a significant amount of low-resolution ones"), without disclosing specific channels, procurement, or licensing arrangements; it is judged to be primarily web crawling + an internal data pool [the exact composition is uncertain]. Confirmable parts:
· The image side explicitly uses two public datasets: LAION-5B and COYO-700M (2B images taken after aesthetic-score filtering).
· The annotation stage borrows public video caption datasets/models: Panda-70M's caption model is used to produce short captions (the paper also criticizes Panda70M, COCO Caption, and WebVid for having captions that are too short and insufficiently comprehensive, and therefore does not use their text directly).
· Part of the caption training data is synthetic: produced via CogVLM's frame-by-frame image captioning + GPT-4 summarization, which constitutes "model-generated synthetic text."
· No mention of licensed/procured data, film/TV footage libraries, or synthetic video data.

### Data Compliance and Provenance (share of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

[Uncertain] Neither the paper nor the open-source repository discusses the proportion of copyright-licensed training data, rights-cleared datasets, content provenance standards (C2PA), or watermarking of generated content. The only relevant information is indirect: the model weights are released under Apache 2.0 (2B) and Zhipu's own open-source license (5B series, permitting commercial use); during cleaning, videos with "obvious subtitles or watermarks" were deliberately removed (the high-quality fine-tuning stage explicitly states it "effectively removed generated subtitles and watermarks"), but the motivation for this action is visual quality rather than copyright compliance. The LAION-5B / COYO-700M datasets used on the image side are themselves public URL-text pair datasets whose copyright status falls within a publicly disputed category, which the paper does not address.

### Clip Duration Distribution and Splitting Strategy ⚠️

· Training clips average about 6 seconds and are entirely single-shot; no duration histogram is given [distribution details uncertain].
· "Mixed-duration training" is explicitly adopted: the paper criticizes fixed-frame-count training for forcing the "discarding of short videos and truncation of long videos," which leads to underutilized data. It therefore places videos of different durations (and correspondingly different resolutions) into the same batch, using a Multi-Resolution Frame Pack (inspired by Patch'n Pack) to ensure consistent shapes within a batch, and then uses 3D RoPE to model the positional relationships among different shapes. This effectively replaces traditional bucketing with packing (the paper explicitly notes that SDXL-style bucketing "makes the data and training pipeline more complex").
· The duration ceiling of the training stages is progressively relaxed: stage1/stage2 max 6 seconds, stage3/stage4(FT) max 10 seconds (corresponding to sequence lengths of 25k → 75k → 700k).
· To save GPU memory, the 3D VAE is trained in two stages: first on 17-frame video, then fine-tuned with context parallelism on 161-frame video.
· Splitting strategy (shot_segmentation): the paper does not describe an explicit scene-splitting tool; instead, a "Lack of Motion Connectivity" classifier is used to entirely discard clips containing splices/transitions.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

· Resolution curriculum (progressive training): first trains at 256px to learn semantics and low-frequency knowledge, then progressively increases to 512px and 768px to learn high-frequency detail. The training hyperparameter table gives the maximum resolution for each stage: stage1 256×384 → stage2 480×720 → stage3 768×1360 → stage4(FT) 768×1360. CogVideoX-5B's final output is 720×480; CogVideoX1.5-5B outputs 1360×768, 10 seconds, 16fps; the product version New Qingying can reach 4K/60fps.
· Aspect ratio: it is explicitly stated that the original aspect ratio is kept unchanged, with only the short side resized to the target resolution ("we keep the aspect ratio unchanged and resize the short side to above resolutions"), thereby preserving the ability to generate videos of arbitrary aspect ratio; CogVideoX1.5-5B-I2V supports arbitrary-resolution output. No landscape/portrait ratio figures are given [uncertain].
· Bucketing vs. packing: does not use SDXL-style bucketing, instead using a Multi-Resolution Frame Pack to pack samples of different resolutions and durations into the same batch.
· When adapting RoPE for high resolution, both interpolation and extrapolation were compared; extrapolation was ultimately chosen to preserve local detail and relative positional relationships.

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

[Uncertain] The paper discloses no information at all about the training data's category/domain distribution, concept clustering, or mixture-balancing strategy — this is the most conspicuous gap in CogVideoX's data engineering relative to Movie Gen, Seedance, etc.: no concept clustering, no long-tail resampling, no target proportion for human presence, no verb/action taxonomy.
The domain tendencies that can be indirectly inferred come from the inverse constraint of the "negative labels":
· "Lecture Type" (lecture/livestream talking-head content, where the frame is essentially static with only a person speaking) is removed as an entire category → "static person talking" content is systematically suppressed in the training set, which also explains the model's weakness in lip-motion/talking-head scenes.
· "Text Dominated" (large amounts of visible text or content dominated by text) is removed as an entire category → image-text, subtitle-heavy, and presentation-slide-style content is excluded.
· "Noisy Screenshots" (raw phone/computer screen recordings) is removed as an entire category → screen recordings, gameplay footage, and UI demos are excluded.
· "Editing" (containing obvious manual editing and effects) is removed → heavily edited music-video- and trailer-style content is excluded.
The dataset is therefore actually skewed toward "un-re-edited, real-world footage with genuinely continuous motion, single-shot material."
· On the captioning side, the GPT-4 summarization prompt explicitly requires coverage of five element categories — "objects, scenery, animals, characters, and camera movements" — which can be seen as content dimensions the team cared about, but this was not used to drive mixture control.
· On the evaluation side, VBench emphasizes dimensions such as Human Action, Scene, Dynamic Degree, Multiple Objects, and Appearance Style — this belongs to evaluation choices rather than data mixture.

### Audio Category Distribution and Mixture (proportions and control strategy for speech/sound-effects-foley/music/ambient sound/silence) — dimension unique to AV models ⚠️

Not applicable to CogVideoX proper: the video model's training entirely does not use an audio track, so the data pipeline contains no audio dimension whatsoever, nor any classification or mixture of speech/sound effects/music/ambient sound/silence.
Audio data does exist on the cascaded CogSound side, but its training data composition, audio-category mixture, and silence-handling strategy have not been disclosed [uncertain]. The only public information states that its generation targets cover complex sound effects such as "explosions, flowing water, musical instruments, animal calls, vehicle sounds," as well as rhythm/musical elements, suggesting its training data is dominated by foley sound effects with some musical component, but there is no quantified mixture. There is also no description of whether it generates or models spoken dialogue.

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

· All training clips are single-shot: the paper explicitly states "approximately 35M single-shot clips," averaging 6 seconds. Multi-shot narrative is outside the modeling scope; the model outputs single-shot continuous video.
· The mechanism ensuring single-shot content is not a scene splitter but a negative-label classifier: "Lack of Motion Connectivity" specifically identifies "motion that is incoherent at transitions, commonly seen in videos manually spliced together or edited from static images," and clips containing shot cuts/splices are entirely judged negative and discarded; the "Editing" label further removes material with transition effects.
· Shot-count distribution and average shot count are not given [uncertain].
· Whether native audio tracks are included: not applicable; the video side discards audio tracks entirely during training.
· At the caption level, camera-movement descriptions are included, but there is no structured multi-shot field.

### Language/Accent Distribution (data foundation for multilingual lip-sync capability) ⚠️

Not applicable and not modeled. CogVideoX does not generate speech; the data pipeline has no ASR, no speaker attributes, no language/accent annotation, and no lip-sync capability (on the contrary, the "Lecture Type" label removes talking-head videos as an entire category, actively weakening the model's data foundation for scenes of people speaking).
On the text-conditioning side: captions and the T5 text encoder are primarily oriented toward English (both the paper and the caption-generation prompt are in English, with a text length cap of 226 tokens); the official repository recommends using English prompts, with Chinese prompts requiring rewriting first. Whether CogSound involves multiple languages of speech is undisclosed [uncertain].

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (number of filtering tiers, order of each tier)

CogVideoX's data pipeline is a three-stage structure of "negative-label classifiers + continuous-score thresholds + dense recaptioning," clear in structure but shallower in depth (much simpler than Movie Gen's ten-plus-tier funnel):
【Tier 1: Negative-label filtering】Manually annotate 20,000 videos with positive/negative labels → train six binary classification filters based on Video-LLaMA → apply batch screening to the full raw video corpus. The six negative-label categories are: Editing (manual re-editing/effects that damage visual integrity), Lack of Motion Connectivity (incoherent motion at transitions, manually spliced or edited from static images), Low Quality (poor filming, unclear picture, severe camera shake), Lecture Type (lecture/livestream talking-head, very little effective motion), Text Dominated (dominated by text), Noisy Screenshots (raw phone/computer screen recordings). The paper summarizes the design motivation as two categories of intrinsic noise (manual editing during video creation distorting genuine dynamic information; image-quality issues from filming equipment and shake) plus one category of training-suitability issue (too little dynamic information or lack of dynamic connectivity).
【Tier 2: Continuous-score dynamic thresholds】Optical flow scores (representing motion intensity) and image aesthetic scores (representing image quality) are computed for all training videos, and these two thresholds are dynamically adjusted during training ("dynamically adjust their threshold during training") to ensure the dynamism and aesthetic quality of generated video. The hyperparameter table gives an aesthetic-score lower bound of 4.5. This practice of "dynamically tightening thresholds as training progresses" is one of the more distinctive aspects of CogVideoX's data engineering.
【Tier 3: Dense recaptioning】For videos that pass the filters, the Dense Video Caption Data Generation pipeline is run to produce long captions (see caption_model / caption_structure for details).
【Tier 4: High-quality subset fine-tuning】The pretraining data still retains dirty data such as subtitles, watermarks, and low bitrate, so in the final training stage a higher-quality 20% subset is selected out for fine-tuning.
【Image branch】LAION-5B and COYO-700M are only aesthetic-score filtered, taking 2B images that are mixed into video training (each image treated as a single-frame video).
【Inference-side alignment】A fine-tuned LLM (GPT-4V / CogVLM for image-to-video) performs prompt upsampling, rewriting the user's short prompt into a long description matching the distribution of training captions.

### Funnel Quantitative Retention Rate (input/output volume and final retention rate at each tier, e.g., Apollo's 27%) ⚠️

[Uncertain] The paper gives no tier-by-tier quantitative funnel (input/output volume and retention rate for each tier). The only two quantifiable points are:
· After filtering, approximately 35M single-shot clips ultimately remain, but the size of the original video pool is not disclosed, so the overall retention rate cannot be back-calculated.
· The retention proportion for the high-quality fine-tuning stage is explicitly given as "accounting for 20% of the total dataset" (a subset of higher quality video data) — this is the only explicit proportion figure in the entire paper.
· Appendix K gives confusion matrices for the six classifiers on the test set (a random 10% of annotated data), from which the proportion of each negative-sample category within the annotation pool can be indirectly inferred: Editing category TP+FN≈0.89 (very high proportion of negative samples), Low Quality category TP+FN≈0.89, Static (motion-incoherent) category TP+FN≈0.52, Lecture category 0.53, Text category 0.62, Screenshot category 0.62; but this is the distribution of the manually annotated sampling pool, not the true data distribution, and cannot be equated with funnel retention rate.

### Shot-Splitting Method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

The paper does not mention the use of PySceneDetect, FFmpeg scene detection, or any explicit shot-boundary detection/splitting tool [the splitting tool is uncertain] — this is a conspicuous gap in its pipeline description.
The mechanism that actually achieves single-shot content is "discriminative removal" rather than "splitting": a dedicated classifier is trained to identify Lack of Motion Connectivity (named Classifier-Static in the main text and in Appendix Table 14, with a test accuracy of 0.92), defined as "clips lacking coherent motion at transitions, commonly seen in videos manually spliced together or edited from static images"; this is combined with an Editing classifier (accuracy 0.91) that removes material with obvious editing and effects. Together, these two ensure that the 35M clips entering the training set are single-shot with continuous motion.
In other words, CogVideoX chose a strategy of "rather discard the whole clip than perform fine-grained splitting," at the cost of lower data utilization, but with the benefit of simple implementation and no error introduced by a splitter.

### Quality Filtering (aesthetic scoring, clarity, OCR text filtering, black-bar/watermark/logo detection)

Quality filtering consists of two lines: "Video-LLaMA discriminators + two continuous scores":
【Discriminator line】Of the six Video-LLaMA binary classifiers, four directly serve image quality and content cleanliness; Appendix K Table 14 gives test-set performance (a random 10% of annotated data):
· Classifier-Low Quality (poor filming quality, blurry image, severe camera shake): TP 0.80 / FP 0.02 / TN 0.09 / FN 0.09, Test Acc 0.89 (lowest of the six)
· Classifier-Editing (manual re-editing and effects): TP 0.81 / FP 0.02 / TN 0.09 / FN 0.08, Acc 0.91
· Classifier-Text (text-dominated, equivalent to a discriminative substitute for OCR text filtering): TP 0.60 / FP 0.03 / TN 0.36 / FN 0.02, Acc 0.96
· Classifier-Screenshot (phone/computer screen recording): TP 0.61 / FP 0.01 / TN 0.37 / FN 0.01, Acc 0.98
· Classifier-Lecture (talking-head lecture): Acc 0.99 (highest of the six)
· Classifier-Static (motion-incoherent): Acc 0.92
【Continuous-score line】Image aesthetic scores are computed for the full video corpus; the hyperparameter table gives a lowest aesthetic value of 4.5 as the floor; the threshold is dynamically adjusted across training stages. The 2B images in the image branch are similarly filtered by aesthetic score.
【Final stage】The 20% subset used for high-quality fine-tuning specifically addresses three categories of dirty data remaining in the pretraining data — "subtitles, watermarks, low-bitrate" — and the paper reports that this step "effectively removed generated subtitles and watermarks and slightly improved visual quality."
Notably, the paper does not describe Movie Gen-style dedicated detectors such as black-bar detection, logo detection, or a standalone OCR model — text filtering is accomplished end-to-end using the Video-LLaMA classifier, representing an early practice of "replacing traditional shallow detectors with a multimodal large model."

### Motion Filtering (optical-flow/motion-score threshold, static and shaky-footage removal) ⚠️

Two mechanisms operate in parallel:
· Optical flow score: computed for all training videos, with the threshold dynamically adjusted during training, aimed at "ensuring the dynamic ... quality of generated videos." The specific threshold values are undisclosed [uncertain].
· Video-LLaMA static/connectivity classifier (Classifier-Static, corresponding to the negative label Lack of Motion Connectivity, test accuracy 0.92): removes clips with very little dynamic information, or with incoherent motion at transitions (manual splicing, static-image editing).
· The "Lecture Type" classifier (Acc 0.99) also, in effect, plays a role in removing near-static footage.
· Shake handling falls on the image-quality line rather than the motion line: severe camera shake is classified under the Low Quality negative label ("excessive camera shake") and removed by Classifier-Low Quality, rather than using a "shots per second" proxy metric for shake, as Movie Gen does.
· The paper does not use FFmpeg-side metrics such as motion vectors or VMAF motion score.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

[Uncertain] The paper does not mention a deduplication step at all — neither exact deduplication (hashing/fingerprinting) nor embedding-based semantic deduplication or near-duplicate detection (such as an SSCD copy-detection embedding), nor semantic clustering and long-tail resampling. This is one of the most conspicuous gaps in CogVideoX's data pipeline relative to contemporaneous work.
The only thing indirectly related to "redundancy" is the Editing and Lack of Motion Connectivity negative labels (which can remove videos spliced repeatedly from the same material), but their design purpose is to ensure motion authenticity rather than deduplication.

### VLM/LLM as Data Quality Inspector (multimodal LLM quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to LLM-based semantic judgment)

CogVideoX is one of the early representatives of the paradigm of "using a multimodal large model as a data quality inspector," and it relies on large models on both the annotation and quality-inspection sides:
【Quality inspection side】The core quality inspectors are six binary video classifiers fine-tuned from Video-LLaMA, rather than traditional shallow scorers (aesthetic CNNs, OCR detectors, static detectors, etc.). Trained on 20,000 manually annotated positive/negative labels, they make end-to-end semantic judgments directly on entire video clips, covering in one pass six dimensions — editing traces, motion coherence, filming quality, content genre (lecture), text proportion, and screen-recording origin — that would otherwise require multiple dedicated detectors. Appendix K reports test accuracy in the range 0.89–0.99 (Low Quality 0.89, Editing 0.91, Static 0.92, Text 0.96, Screenshot 0.98, Lecture 0.99). This approach of "one VLM assigning six semantic labels" is closer to the mainstream form of 2026 than contemporaneous approaches that relied on combinations of LAION aesthetic scores + OCR + static detectors.
【Annotation side】GPT-4 serves as the teacher to generate summary-style captions, CogVLM serves as the frame-by-frame image recaptioning model, ultimately distilled down into LLaMA2 (for summarization acceleration) and CogVLM2-Caption (for end-to-end video annotation).
【Limitations】Large-model judgment is still primarily used for "binary-classification filtering," not for producing continuous quality scores, not for semantic-consistency verification of caption-video mismatch, and not for automated review of generated results; continuous scores are still provided by traditional optical-flow and aesthetic models. It can therefore be characterized as a transitional form in the shift "from shallow scorers to large-model semantic judgment."

### Safety and Compliance Filtering (NSFW, copyright, faces/privacy) ⚠️

[Uncertain] Neither the paper nor the open-source repository describes the training-data-level NSFW, violence, copyright, or face/privacy filtering processes and tools, nor is any generation-side safety model or watermarking scheme disclosed. The only relevant clues are indirect: the Text Dominated and Noisy Screenshots negative labels incidentally remove some material containing sensitive text or screen content; the open-source licenses (Apache 2.0 / Zhipu's own open-source license) contain standard compliance-use clauses; as a domestically operated public service, the Qingying product must comply with the "Interim Measures for the Management of Generative Artificial Intelligence Services" and complete algorithm filing, but technical details are undisclosed.

## Annotation Methods

### Caption Model Used (in-house VLM/open-source model, model scale)

Two lines constitute a "teacher generation → student distillation" annotation system:
【Offline teacher pipeline (Dense Video Caption Data Generation, Figure 7)】
1) Panda-70M's video caption model → first generates a short caption;
2) CogVLM (i.e., the image recaptioning model used in CogView3) → samples 1 frame every 2 seconds and generates a dense image caption for each frame;
3) GPT-4 → summarizes the timestamped image_captions dictionary into the final video caption (Appendix G gives the full prompt, requiring content and changes to be described in chronological order, covering objects, scenery, animals, characters, and camera movements, and explicitly prohibiting stock phrases such as "The video presents / depicts / showcases," "throughout the video," and line breaks);
4) For acceleration, 50,000 GPT-4 summary data points were collected to fine-tune a LLaMA2 model as a substitute summarization model for GPT-4, enabling large-scale production.
【Online student model】CogVLM2-Caption: built on CogVLM2-Video + Llama3 as its base, fine-tuned on dense caption data produced by the above pipeline into an end-to-end video understanding/annotation model, at roughly the 12B scale (a Llama3-8B-series base plus the visual side), used to further accelerate full-corpus recaptioning. This model has been open-sourced (huggingface.co/zai-org/cogvlm2-llama3-caption), the only component in this work's data pipeline that can be directly reused externally. The paper also found that chaining CogVLM2-Caption with CogVideoX enables video-to-video generation (Appendix I), which indirectly demonstrates that its captions capture almost all the detail of the original video.
【Inference side】There is also a caption upsampler: a fine-tuned LLM (for text-to-video) or GPT-4V / CogVLM (for image-to-video) rewrites the user's short prompt into a long description in the style of the training captions; the paper explicitly states that the fine-tuned LLM outperforms zero-shot/few-shot approaches.

### Caption Density and Structuring Degree (short/long/dense descriptions, structured fields such as camera movement, style tags)

· Density: explicitly pursues dense, long captions. The paper criticizes existing datasets such as Panda70M, COCO Caption, and WebVid for having captions that are "usually very short and cannot comprehensively describe the video," and therefore builds its own pipeline to produce paragraph-level long descriptions. The comparison example in Appendix H is highly representative: for the same video, Panda-70M outputs "A crab is walking on the beach with a light bulb on its back." (a single sentence), while CogVLM2-Caption outputs a paragraph of nearly 80 words, including subject appearance (dark glossy shell, reddish-brown legs), action, lighting change (from a soft glow to a more pronounced illumination), setting (sandy terrain, tranquil sea backdrop), time (at night), and atmospheric evaluation (serene yet whimsical atmosphere).
· Structuring degree: belongs to the category of "unstructured dense natural-language paragraphs," with no Movie Gen-style explicit fields (no independent camera-movement label prefix, no FPS token, no enumerated style tags). However, the GPT-4 summarization prompt enforces constraints on content coverage and writing style, forming a de facto implicit schema: it must cover five categories of elements — objects / scenery / animals / characters / camera movements; it must describe content and its changes in chronological order; and it prohibits opening with fixed stock phrases to avoid stylistic collapse.
· Length constraint: the text encoder (T5)'s Text Length cap of 226 tokens is the practical ceiling on caption length.
· Training-inference distribution alignment: because users' actual input is far shorter than training captions, a dedicated caption upsampler is designed to rewrite short prompts into long descriptions at inference time, consistent with the approach used by DALL·E 3.

### Joint Audio-Video Caption Structure (whether visual + auditory tracks are covered simultaneously, whether they are split into separate fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field structure) ⚠️

Not applicable. CogVideoX's captions are purely visual descriptions, not touching the auditory track at all; the training data contains no audio track, so there is no joint audio-video caption schema, nor any visual/auditory stream split.
Whether the cascaded CogSound uses text captions as a condition, and the structure of any audio caption, is undisclosed [uncertain]; from public descriptions, CogSound is primarily conditioned on video features (GLM-4V understanding + frame-level feature cross-attention), and the role of text conditioning is not described.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

None. CogVideoX's data pipeline has no ASR transcription and no speaker identity/language/accent/emotion annotation, and the model does not generate speech. On the contrary, the team actively lists "Lecture Type" (dominated by a person continuously speaking, with very little effective motion) as a negative label and removes it as an entire category using a classifier with 0.99 accuracy, effectively excluding dialogue-dense material at the data level. Whether CogSound processes speech is undisclosed [uncertain].

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action labels, explicit state)

There is essentially no explicit geometric or structured annotation: the paper does not use camera intrinsics/extrinsics, depth maps, 3D point tracks, optical-flow fields (optical flow is used only as a scalar score for filtering, not retained as an annotation), segmentation masks, or action-category labels.
The only structural element is implicit language annotation: the GPT-4 summarization prompt explicitly requires captions to include "camera movements," so camera-movement information is embedded in captions as natural language rather than as an independent categorical field or label taxonomy (in contrast to Movie Gen's 16-class camera-movement classifier + prefix concatenation).
"Structuring" on the model side is mainly reflected in positional encoding rather than data annotation: 3D RoPE explicitly models positional relationships across the three dimensions of text-time-space, and the Multi-Resolution Frame Pack carries each sample's shape information within a batch.

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV) ⚠️

· Visual data: entirely real captured video; no synthetic video is constructed, and no controlled perturbation or editing-based training pairs are constructed.
· Text data: captions are entirely synthetic text, and form a multi-level model synthesis chain — Panda-70M short caption + CogVLM frame-by-frame image caption → GPT-4 summarization (50,000 items) → distilled into LLaMA2 → further distilled into CogVLM2-Caption. This is the only systematic synthetic-data construction in this work, and it belongs to "synthetic annotation" rather than "synthetic samples."
· Inference-side synthesis: the caption upsampler uses an LLM to synthesize long prompts, aligning the inference input distribution closer to the training caption distribution — essentially also a form of synthetic data alignment.
· Details of training-pair construction for the image-to-video (I2V) model are given in Appendix D; the main text does not describe whether synthetic first frames are used [uncertain].

### Degree of Human Involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

Human involvement is concentrated at one point — "seed annotation for training the filters" — and the overall pattern is the classic paradigm of "a small amount of manual annotation → train a model → fully automatic large-scale screening":
· Filter seed annotation: 20,000 videos were manually sampled and annotated with positive/negative quality labels (covering the six negative-label categories), with a random 10% held out as a test set used to report classifier accuracy. This is the only clearly identified manual annotation step in the entire pipeline.
· The negative-label taxonomy itself was manually designed (the paper lists the definitions of the six categories and gives example videos for each in Figure 16).
· No manual work in the annotation stage: captions are entirely generated automatically by the model chain, with no mention of manual polishing or review (in contrast to Movie Gen, where annotators rewrite captions line by line at the SFT stage).
· Whether the selection criteria for the high-quality fine-tuning subset (20%) were automatic scoring or manual screening is not stated [uncertain]; based on the wording, automatic threshold-based screening is inferred.
· There is heavy manual involvement on the evaluation side: 100 carefully designed prompts, with human reviewers assigning fine-grained scores on a 0–1 scale and an overall score on a 0–5 scale, with the rule that "if instructions are not followed, the overall score cannot exceed 2." Evaluation dimensions include Sensory Quality, Instruction Following, Physics Simulation, and cover-image quality, each with detailed three-tier (1 / 0.5 / 0) scoring rubrics (Appendix J). The acknowledgments specifically thank the data annotators.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment) ⚠️

There is no audio-video alignment step on the CogVideoX video side (audio tracks are not used).
Alignment on the CogSound side is a mechanism at the modeling level rather than a detection step at the data-filtering level:
· Block-wise Temporal Alignment Cross-attention: by learning the relationship between frame-level video features and audio features, it precisely connects video and audio features, ensuring consistency between audio and video at both the temporal and semantic levels.
· Rotary position encoding (RoPE): provides a unique identifier for each position in the audio sequence and captures relative relationships, improving temporal consistency and stability for long-sequence tasks.
· Video understanding based on GLM-4V: first accurately identifies the semantics and emotion behind the video, then generates matching audio accordingly — this is semantic-level alignment.
Whether the data side performed synchronization detection and filtering (lip sync, event alignment) is entirely undisclosed [uncertain]. CogSound is explicitly positioned as sound-effects generation rather than dialogue dubbing, and there is no lip-sync-related capability or metric.

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Uncertain] No public synchronization detection metrics or thresholds exist. Not applicable to the CogVideoX video side; CogSound has no technical report, and it is undisclosed whether metrics such as SyncNet / AV-align / ImageBind score are used, nor is there any confidence threshold value. What can be confirmed is that no SyncNet-type lip-sync metric is used anywhere in the pipeline (the model does not generate speech).

### Separate Handling of Temporal Sync vs. Semantic Sync (temporal alignment and content-semantic matching as two independent filtering conditions) ⚠️

Not applicable on the video side. CogSound's public description conceptually distinguishes two levels — Block-wise Temporal Alignment Cross-attention is responsible for "temporal-level consistency," while GLM-4V-based semantic/emotional understanding is responsible for "semantic-level matching"; the official description states it "ensures consistency between audio and video at both the temporal and semantic levels," which can be seen as a separate treatment of temporal alignment and content-semantic matching — but this is reflected in the model's architectural design rather than being applied as two independent data-filtering conditions [whether the data side treats these separately is uncertain].

### Audio Quality Filtering (SNR, silence detection and silence-ratio threshold, no-audio-track removal, off-screen-source removal, background-music separation) ⚠️

[Uncertain] The CogVideoX video side does not involve audio at all; the audio track is directly discarded during training, with no SNR, silence detection, no-audio-track removal, off-screen-source removal, or BGM separation processing. CogSound's audio quality filtering strategy (SNR threshold, silence ratio, vocal separation, etc.) is undisclosed.

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

[Uncertain] Not applicable to the CogVideoX video side. From public descriptions, CogSound appears to be a "single model generating multiple audio categories" — capable of generating complex sound effects such as explosions, flowing water, musical instruments, animal calls, and vehicle sounds, and can also generate musical elements such as rhythm, but there is no description of separate models or separate data streams by category of speech/sound effects/music, nor any classifier, mixture ratio, or differentiated handling strategy per category.

## Training Integration

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

The training curriculum follows the main line of "progressive resolution and duration + final high-quality fine-tuning," with the hyperparameters of the four stages given in full in Table 5 of the paper (shared by CogVideoX-2B and 5B):
· stage1: max resolution 256×384, max duration 6s, batch size 2000, sequence length 25k, 400k training steps — learning semantics and low-frequency knowledge;
· stage2: 480×720, 6s, batch 1000, sequence 75k, 220k steps;
· stage3: 768×1360, 10s, batch 250, sequence 700k, 120k steps — learning high-frequency detail;
· stage4 (FT): 768×1360, 10s, batch 100, sequence 700k, 10k steps — high-quality fine-tuning.
It can be seen that as resolution increases, batch size drops from 2000 to 100, sequence length rises from 25k to 700k, and step count drops from 400k to 10k — compute is heavily tilted toward the low-resolution stages.
Other curriculum elements:
· Mixed image-video training runs throughout (each image treated as a single-frame video), resolving the problem of a model splitting into two generation modes by token count under fixed-frame-count joint training;
· Mixed-duration training + Multi-Resolution Frame Pack allows samples of different durations/resolutions to share a batch, avoiding the discarding of short videos and truncation of long videos;
· Data filtering thresholds (optical flow score, aesthetic score) are dynamically adjusted during training, i.e., the data curriculum is coupled to the training progress;
· The 3D VAE has its own two-stage curriculum: first trained on 17 frames, then fine-tuned with context parallelism to 161 frames;
· The diffusion setup adopts v-prediction + zero SNR, and proposes Explicit Uniform Sampling (dividing 1..T into n intervals, with each rank sampling uniformly within its own interval) to stabilize the loss curve and accelerate convergence;
· When adapting RoPE for high resolution, extrapolation is chosen over interpolation.

### Data Mixture Changes Across Training Stages (pretraining/annealing/SFT high-quality subset) ⚠️

· Resolution dimension: as stages progress, training data switches progressively from 256px to 512px to 768px (keeping the aspect ratio and only resizing the short side); low-resolution data dominates absolutely in the early stages.
· Duration dimension: stage1/2 caps at ≤6s, stage3/4 relaxes to ≤10s; longer clips only enter in later stages.
· Quality dimension: the thresholds for optical flow score and aesthetic score are dynamically tightened during training ("dynamically adjust their threshold during training"), forming an implicit data curriculum of "quality monotonically improving with stage."
· Annealing/fine-tuning stage: the final stage (stage4 FT, 10k steps) uses only the highest-quality 20% subset, specifically targeting the removal of subtitles, watermarks, and low-bitrate data. The paper honestly reports the cost: this step "effectively removed generated subtitles and watermarks and slightly improved visual quality, but at the same time a slight degradation in the model's semantic ability was observed" — this is a rare piece of negative evidence regarding "high-quality small-data fine-tuning sacrificing semantic coverage."
· Image-text mixture ratio: 2B images and 35M video clips are trained together; the specific batch-level mixture ratio is not disclosed [uncertain].

### Post-Training Data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

· Post-training takes the form of high-quality SFT only: a higher-quality 20% subset is selected from the pretraining data and trained for 10k steps at stage4 (batch 100, 768×1360, 10s). The selection criteria target three categories of dirty data — subtitles, watermarks, low bitrate — with specific thresholds and whether manual involvement occurred undisclosed [uncertain].
· No preference data is used: the paper contains no preference-pair annotation, no reward model, and no RLHF/DPO/RLAIF anywhere. Human evaluation (100 prompts, 0–5 scale, multi-dimensional scoring rubric) is used only for evaluation, not for training. This stands in sharp contrast to the RLHF-heavy approaches of contemporaneous and later models such as Seedance and Kling.
· The I2V model is obtained by continuing to train on top of the T2V base (Appendix D); the details of constructing its paired data are not disclosed in the main text [uncertain].
· The post-training data increment of CogVideoX1.5 relative to 1.0 is undisclosed [uncertain].

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

[Uncertain] The paper does not disclose the infrastructure, framework (no mention of NeMo Curator, Data-Juicer, etc.), GPU acceleration ratio, unit throughput, or cost on the data-processing side. The inferable toolstack: Video-LLaMA (the six filtering classifiers), an optical-flow computation module, an image aesthetic-scoring model, the Panda-70M caption model, CogVLM, the GPT-4 API, LLaMA2, and CogVLM2-Video/Llama3 — of which the two distillation steps ("using LLaMA2 to distill GPT-4 summaries" and "using CogVLM2-Caption to end-to-end replace the three-stage pipeline") both have their motivation explicitly stated as "accelerating large-scale generation (to accelerate this process / to further accelerate video recaptioning)," direct evidence of throughput optimization on the data-processing side, but no acceleration multiplier or processing-scale figures are given.
Training-side infrastructure disclosure is likewise limited: the paper gives the batch size, sequence length, and step count for each stage, with the maximum sequence length of 700k requiring context parallelism; VAE training uses context parallelism extended to 161 frames; on the inference side, timing and memory on H800 are given (5B at 480×720×6s takes 113s / 26GB, 5B at 768×1360×5s takes 500s / 76GB), but the training cluster scale and GPU-hours are not disclosed [uncertain].

## Performance Comparison

### Quantified Impact of Data-Strategy Ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper's ablation experiments focus mainly on model architecture and training techniques, with very few quantified ablations on the data side — a clear shortcoming relative to Movie Gen:
【Architecture/training ablations (not data)】3D RoPE vs. sinusoidal absolute position encoding (RoPE loss converges significantly faster); Expert AdaLN vs. MMDiT vs. no Expert AdaLN (by FVD, CLIP4Clip Score, and loss, Expert AdaLN is superior); 3D full attention vs. separated 2D+1D attention; Explicit Uniform Sampling (loss curve noticeably more stable, converges faster); RoPE extrapolation vs. interpolation (extrapolation preserves local detail, interpolation produces blurry large images).
【Qualitative conclusions directly related to data】
· Caption density ablation: the paper claims that "our innovative video captioning model significantly improves generation quality and semantic alignment," and in Appendix H demonstrates the density difference via a comparison between Panda-70M's short caption and CogVLM2-Caption's long caption, but gives no A/B net-win-rate or metric-increment figures [quantitative result uncertain].
· High-quality fine-tuning ablation: explicitly reports a two-sided effect — removal of subtitles/watermarks and a slight improvement in visual quality, but a slight degradation in semantic ability. This is a qualitative but highly valuable conclusion about the tradeoff of "filtering strictness."
· Mixed-duration training / Frame Pack: the motivation argued is that fixed-frame-count training causes a model to split into two generation modes by token count and generalizes poorly, but no comparison figures are given [quantitative result uncertain].
· Data-mixture ablation: not performed [uncertain].

### Evidence for Quality vs. Quantity (cases where small, curated data outperforms large, messy data) ⚠️

Evidence exists but is less thorough than in contemporaneous work, and the paper's own stance leans toward "scale and quality both matter":
· Evidence supporting quality-first: (1) strict up-front filtering using six Video-LLaMA classifiers + dual optical-flow/aesthetic thresholds, preferring to discard entire clips rather than perform fine-grained splitting and repair, ultimately retaining only 35M single-shot clips; (2) the final stage uses only a 20% high-quality subset for 10k steps of fine-tuning (less than 1/40 of stage1's 400k steps), and this alone effectively removes subtitles and watermarks from generated results and improves visual quality; (3) caption quality is listed by the paper as one of the key factors in improving generation quality and semantic alignment, i.e., "for the same video, better text annotation yields a better model."
· Counter-evidence (honestly recorded by the paper): high-quality fine-tuning, while improving visual quality, caused "a slight degradation in the model's semantic ability," indicating that over-tightening data quality can sacrifice semantic coverage and concept diversity — a direct observation of the cost of the "small and curated" approach, relatively rare among comparable papers.
· The paper's overall stance leans toward scaling: the conclusion explicitly states "CogVideoX has scalability, and as the size of model parameters, data volume, and training volume increase, the performance will get better in the future," and does not advocate for a small-data approach.
· No controlled experiment is given showing "small, curated data surpassing large, messy data" [uncertain].

### Alignment Between Training Data Domain Distribution and Evaluation Benchmark Taxonomy (e.g., VABench's seven major categories)

The alignment relationship is weak and passive — CogVideoX adopts an external general-purpose benchmark rather than building its own taxonomy, and there is no domain-mixture strategy on the training-data side to align in the first place:
· Automated evaluation uses VBench (excluding quality-irrelevant sub-metrics such as color and bitrate), as well as Dynamic Quality (proposed by Devil) and CLIP4Clip, among others; the dimensions the paper focuses on are those "consistent with human perception," such as Human Action, Scene, Dynamic Degree, Multiple Objects, and Appearance Style, presented via a radar chart.
· The training data has no corresponding category-mixture control (no concept clustering, no long-tail resampling, no target proportion for people), so there is no one-to-one correspondence between a "training taxonomy" and a "benchmark taxonomy" of the kind seen in Movie Gen.
· The only identifiable alignment is "negative": on the data side, the three negative labels Lecture Type, Text Dominated, and Noisy Screenshots actively exclude static talking-head, text-dominated, and screen-recording content, which aligns directionally with the evaluation side's emphasis on Dynamic Degree / Human Action — i.e., data filtering is aligned with evaluation goals along the dynamism dimension.
· Human evaluation builds its own four-dimensional scoring system (Sensory Quality, Instruction Following, Physics Simulation, cover-image quality; 100 prompts, 0–5 scale, with the overall score capped at 2 if instructions are not followed), and compares against Kling, with CogVideoX-5B winning in human evaluation; this system has no direct mapping to the training data's category distribution.
· No VABench-style joint audio-video taxonomy is adopted (the model has no audio dimension).

## Uncertain Fields

Research information for the following fields is partially uncertain (sources marked with ⚠️):

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
- motion_filtering
- deduplication
- safety_filtering
- joint_av_caption_schema
- dialogue_transcription_attributes
- synthetic_data_synthesis
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
