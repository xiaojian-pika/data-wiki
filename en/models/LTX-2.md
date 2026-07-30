# LTX-2 (including the subsequent LTX-2.3; technical report "LTX-2: Efficient Joint Audio-Visual Foundation Model")

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

LTX-2 (including the subsequent LTX-2.3; technical report "LTX-2: Efficient Joint Audio-Visual Foundation Model")

### Releasing institution/company

Lightricks (Israel; the team behind LTX Studio / the LTXV model series)

### Release date (technical report/paper/open-source date)

LTX-2 was first publicly announced in October 2025 (preview/productized integration into LTX Studio); on January 6, 2026 the full set of model weights and training code was officially open-sourced, and the technical report was published on arXiv the same day as arXiv:2601.03233v1 (cs.CV, 14 pages plus 2 pages of supplementary material); on March 5, 2026 the upgraded LTX-2.3 (22B, LTX Desktop) was released. Predecessor foundations: LTX-Video 2B (November 2024), LTXV-13B (May 2025).

### Type (model/dataset/toolchain/evaluation benchmark)

A model (an open-source foundation model for joint text-to-"audio+video" generation, T2AV), accompanied by an open-source toolchain (ltx-core / ltx-pipelines / ltx-trainer inference and training/fine-tuning code, ComfyUI and Diffusers integration, and control LoRAs for camera/pose/lip dubbing, etc.). Not a dataset, not an evaluation benchmark.

### Degree of openness (whether weights/code/data/pipeline are each open source)

Its degree of openness is the highest among peer joint audio-video generation models, but the data side remains closed.
【Open-sourced】(1) All model weights: ltx-2-19b-dev (bf16, trainable), fp8/fp4 quantized versions, the 8-step distilled version ltx-2-19b-distilled, as well as the spatial/temporal upsamplers, all released on Hugging Face (Lightricks/LTX-2, with roughly 427K downloads per month); (2) inference code and multi-backend integration; (3) training/fine-tuning code ltx-trainer (supporting LoRA, full-parameter fine-tuning, IC-LoRA); (4) the technical report discloses architectural details publicly.
【Not open-sourced】the training data itself, the data-processing pipeline code, the internal captioner model, the aesthetics scoring model, and data statistics figures.
【License】LTX-2 Open Weights License: free for academic research; free for commercial use by companies with annual recurring revenue (ARR) under $10 million; companies above this threshold must obtain a commercial license from Lightricks. The company officially describes it as "the first truly open-weight, production-grade audio-video generation model."

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Supported, and implemented as native joint generation — this is the model's core positioning. The implementation approach is a "decoupled yet integrated asymmetric dual-stream" design, which is neither cascaded nor MoE-fused:
(1) Each modality has its own independent VAE: video uses a spatiotemporal causal VAE; audio is first converted into a 16kHz stereo mel spectrogram (the two channels concatenated along the channel dimension) and then passed through an independent causal audio VAE, where each latent token corresponds to about 1/25 second of audio at 128 dimensions; after decoding, a modified HiFi-GAN vocoder (with the channel count doubled to support stereo) upsamples and reconstructs a 24kHz two-channel waveform.
(2) Asymmetric dual-stream DiT: a 14B-parameter video stream + a 5B-parameter audio stream (19B combined), sharing the same depth. Each dual-stream block executes sequentially: same-modality self-attention → text cross-attention → audio-video cross-attention → FFN, with RMSNorm between layers. The video stream uses 3D RoPE, the audio stream uses 1D temporal RoPE.
(3) Cross-modal interaction: bidirectional audio-video cross-attention runs throughout the full depth; the cross-modal attention uses only the temporal component of RoPE (forcing attention to focus on temporal synchronization rather than spatial alignment); a cross-modality AdaLN is introduced — the scale/shift of one modality is determined by the hidden state of the other modality together with the diffusion timestep, used to regulate how much cross-modal information is absorbed at each stage.
(4) Modality-aware CFG at inference time (Bimodal CFG): \hat{M}=M(x,t,m)+s_t(M(x,t,m)-M(x,∅,m))+s_m(M(x,t,m)-M(x,t,∅)), where the text guidance s_t and the cross-modal guidance s_m are adjusted independently; the paper uses s_t=3, s_m=3 for the video stream and s_t=7, s_m=3 for the audio stream. Increasing s_m improves temporal synchronization and semantic consistency.
(5) The decoupled latents naturally support V2A (adding synchronized audio to an existing video) and A2V (driving video generation from an audio track) editing workflows.
(6) Output capability: up to 20 continuous seconds of audio-video (exceeding Veo 3's 12 seconds, Sora 2's 16 seconds, and Ovi's 10 seconds); the product-level claim is native 4K at up to 50fps; the inference strategy described in the paper is multi-scale, multi-tile: first generate a base latent at roughly 0.5MP low resolution to establish the global composition/motion/audio-visual synchronization → upsample the latent to raise spatial resolution → refine to 1080p via overlapping spatiotemporal tiles that are then fused in latent space. (The 4K/50fps figures come from product and press-release framing; the body of the paper only goes up to Full-HD 1080p.)

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

- 【Official first-hand】LTX-2: Efficient Joint Audio-Visual Foundation Model, Yoav HaCohen et al., Lightricks, arXiv:2601.03233v1, 2026-01-06 (Section 5 Training Data and Section 5.1 Captioning constitute all first-hand information on the data side): https://arxiv.org/abs/2601.03233
- 【Official first-hand】LTX-2 paper PDF: https://arxiv.org/pdf/2601.03233
- 【Official first-hand】LTX-2 paper HTML version: https://arxiv.org/html/2601.03233v1
- 【Official first-hand · same-team corroboration】LTX-Video: Realtime Video Latent Diffusion, arXiv:2501.00103 (Section 3, Data Preparation, includes a 9-level cleaning pipeline diagram, the aesthetic Siamese model, CLIP-based deduplication, motion filtering, re-captioning, as well as Fig.13's word cloud and Fig.14's distributions of caption word count and clip duration — the LTX-2 dataset is a subset of this dataset): https://arxiv.org/abs/2501.00103
- 【Official first-hand】LTX-2 official GitHub repository (ltx-core / ltx-pipelines / ltx-trainer, LoRA, license): https://github.com/Lightricks/LTX-2
- 【Official first-hand】LTX-2 license text: https://github.com/Lightricks/LTX-2/blob/main/LICENSE
- 【Official first-hand】Hugging Face model card Lightricks/LTX-2 (variants, quantized versions, distilled version, upsamplers, Limitations): https://huggingface.co/Lightricks/LTX-2
- 【Official first-hand】Hugging Face Lightricks/LTX-2.3 (22B upgraded version): https://huggingface.co/Lightricks/LTX-2.3
- 【Official first-hand】Lightricks open-sources LTX-2 press release, GlobeNewswire, 2026-01-06 (4K, open weights and training code, $10M ARR licensing threshold): https://www.globenewswire.com/news-release/2026/01/06/3213304/0/en/Lightricks-Open-Sources-LTX-2-the-First-Production-Ready-Audio-and-Video-Generation-Model-With-Truly-Open-Weights.html
- 【Official first-hand】LTX official newsroom: LTX-2 full weights open-source announcement: https://ltx.io/newsroom/ltx-2-is-now-open-source-full-model-weights-released
- 【Official first-hand】LTX-2 official prompting guide (prompt structure for audio/dialogue/accent/foley/ambience, isomorphic to the training captions): https://ltx.io/model/model-blog/prompting-guide-for-ltx-2
- 【Official first-hand】LTX-2.3 official prompt guide: https://ltx.io/model/model-blog/ltx-2-3-prompt-guide
- 【Official first-hand】LTX official documentation, Prompting Guide: https://docs.ltx.io/api-documentation/implementation-guides/prompting-guide
- 【Official first-hand · compliance corroboration】Press release on the Lightricks–Shutterstock video training-data partnership, PR Newswire, 2024-12 (first global partner to adopt a research license): https://www.prnewswire.com/news-releases/lightricks-partners-with-shutterstock-for-video-training-data-to-advance-open-source-ltxv-video-ai-generative-video-model-302331526.html
- 【Official first-hand · compliance corroboration】Shutterstock investor-relations page, same press release: https://investor.shutterstock.com/news-releases/news-release-details/lightricks-partners-shutterstock-video-training-data-advance
- 【Official first-hand】Press release on Lightricks' release of the 13B LTX Video model, PR Newswire, 2025-05 (mentions the Getty Images strategic partnership): https://www.prnewswire.com/news-releases/lightricks-launches-13b-parameters-ltx-video-model-breakthrough-rendering-approach-generates-high-quality-efficient-ai-video-30x-faster-than-comparable-models-302447660.html
- 【Third-party report】Wikipedia: LTX-2 (timeline, specifications, leaderboard performance, limitations): https://en.wikipedia.org/wiki/LTX-2
- 【Third-party report】Coverage of the Lightricks 13B LTX Video and Getty partnership, Dataconomy, 2025-05: https://dataconomy.com/2025/05/14/lightricks-unveils-13b-ltx-video-model-for-hq-ai-video-generation/
- 【Third-party report】Lightricks–Shutterstock partnership and AI training-data licensing standards, Metaverse Post: https://mpost.io/lightricks-shutterstock-partnership-sets-new-standards-for-ai-training-data-licensing/
- 【Third-party report】LTX-2's native 4K and fully open weights, Open Source For You, 2026-01: https://www.opensourceforu.com/2026/01/ltx-2-from-lightricks-delivers-native-4k-audio-video-with-fully-open-weights/
- 【Third-party report】LTX-2 technical breakdown, Introl Blog, 2026: https://introl.com/blog/ltx-2-audiovisual-diffusion-synchronized-video-audio-2026
- 【Third-party compilation】awesome-ltx2 prompt guide (dialogue quotation marks, language/accent conventions, foley/ambience/music description norms): https://github.com/wildminder/awesome-ltx2/blob/main/LTX2-prompt-guide.md
- 【Third-party report】Interpretation of the LTX-2 license and scope of commercial use, WaveSpeed Blog: https://wavespeed.ai/blog/posts/blog-ltx-2-license-commercial-use/
- 【Third-party report】LTX-2.3 update highlights (released 2026-03-05, 22B, 4K/50fps/20 seconds), WaveSpeed Blog: https://wavespeed.ai/blog/posts/ltx-2-3-whats-new-2026/
- 【Related-work comparison】Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation, arXiv:2510.01284 (LTX-2's primary open-source point of comparison): https://arxiv.org/abs/2510.01284
- 【Related-work comparison】Veo 3 technical report (LTX-2's closed-source point of comparison): https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf
</content>

## Data Scale and Distribution

### Training data magnitude (video count/hour count/token count, pretraining and SFT listed separately) ⚠️

Entirely undisclosed. Section 5, "Training Data," of the technical report totals only two paragraphs (about 150 words), giving no video count, total duration, or token count, and does not distinguish pretraining scale from fine-tuning scale. It states only qualitatively that the model uses "a subset of the same dataset employed in LTX-Video," a subset focused on "video clips that contained significant and informative audio components." The predecessor LTX-Video paper likewise gives no data-scale figures. [Uncertain]

### Data source composition (proprietary/public datasets/web crawling/licensed acquisition/synthetic data)

The composition is a mixture of "publicly available data + licensed acquired material." The original LTX-Video paper states: "Our training dataset comprises a robust collection of publicly available data, supplemented with licensed material." LTX-2 directly reuses the audio-informative subset of this same data pool.
The licensed sources have clear public corroboration: (1) Shutterstock — announced in December 2024, Lightricks is the world's first partner to train an open-source model using Shutterstock's "research license," gaining access to its HD and 4K video libraries; (2) Getty Images — a strategic partnership established in 2025 during development of the 13B model, providing access to a high-quality video library. In addition, image datasets were mixed into training (LTX-Video explicitly treats images as one "resolution-duration combination" participating in training, used to introduce concepts uncommon in video data). No mention of using synthetic data. The proportion contributed by each source is not disclosed.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Among peer models, Lightricks has the strongest compliance narrative, but quantitative disclosure is lacking. The official and media framing is that "the training data comes entirely from licensed sources (Getty Images and Shutterstock), with no copyright concerns for commercial use"; Shutterstock's "research license" is described as the industry's first instance of lowering the barrier to acquiring training data for open-source models. However, note: (1) the original LTX-Video paper actually says "publicly available data supplemented with licensed material," which diverges from the media framing of "entirely licensed" — the proportion of licensed data has never been published; (2) the technical report gives no list of rights-cleared datasets; (3) there is no mention of C2PA, watermarking, or any output-side provenance mechanism; (4) the report's "Social Impact" section only qualitatively acknowledges that the model will reflect biases present in the training data, and lists "authenticity verification and provenance improvements" as future work. [Uncertain]

### Clip duration distribution and splitting strategy ⚠️

Not separately disclosed for LTX-2. One can refer to the same-source LTX-Video: paper Fig.14(b) gives a histogram of the post-filtering clip duration distribution, ranging roughly 0–30 seconds, with quality concentrated in shorter clips (the distribution decreases monotonically with duration). Splitting strategy: the input unit of the data pipeline is itself "Input Shots," with the entire pipeline operating at shot granularity and producing "Final Shots" as the final output. Training is conducted simultaneously across multiple "resolution × duration" combinations, with resizing used to make the token count roughly equal across samples. The generation-side ceiling is 20 seconds; beyond roughly 20 seconds, temporal drift and synchronization degradation appear. Specific bucketing values are not published. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

No distribution statistics are given, but the strategy is clear. (1) Joint multi-resolution, multi-duration training: training occurs simultaneously across many width/height/duration combinations, and the model generalizes well to unseen configurations; (2) token-count alignment rather than padding/packing: the original video is resized to a comparable token count, and 0%–20% random stochastic token dropping is applied so that each sequence has a fixed token count — the authors state that this approach is simpler and more efficient than complex token-packing/padding while preserving data diversity; (3) aspect-ratio normalization: the pipeline explicitly includes a "Crop Black Bars" step to remove black borders, standardizing aspect ratio and increasing the effective visual area; (4) the pipeline includes a "Resize Shots" step. At inference time, resolution must be divisible by 32 and the frame count must satisfy 8n+1. The specific mixture of resolutions/aspect ratios is not disclosed. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

No category/domain mixture figures are disclosed, nor is any concept-balancing strategy described. There is only one qualitative statement: the selected subset "provides a balanced distribution of visual and auditory content," enabling captions to adequately cover both the image domain and the auditory domain simultaneously. Indirect clues: (1) LTX-Video's paper Fig.13 gives a caption word cloud, which roughly reflects the concept distribution but with no numerical values; (2) image datasets were mixed into training specifically to supplement "concepts uncommon in video data," indicating the team pays attention to concept coverage; (3) the licensed sources are Shutterstock/Getty commercial libraries, whose distribution is naturally skewed toward professionally shot general-purpose stock footage rather than UGC. The limitations section acknowledges that "languages and dialects underrepresented in the training data" lead to degraded performance, indirectly indicating a long-tail distribution issue with no dedicated balancing performed. [Uncertain]

### Audio category distribution and mixture (proportion and control strategy for speech/sound-effect foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

This is the single most valuable — and the only clearly stated — filtering criterion on the data side for LTX-2, yet it still comes with no proportion figures.
【Filtering strategy】Rather than indiscriminately taking the full volume of LTX-Video's videos, the approach "focuses on video clips that contained significant and informative audio components" — i.e., "audio informativeness" is used as the core gate for constructing the AV training subset, excluding clips that are silent, dominated by silence, or whose audio track bears no informative relationship to the visuals. This "audio-informativeness filtering" is the key incremental step LTX-2 adds on the data side relative to purely visual models.
【Coverage categories】The captioning system and the model's capabilities cover four categories: dialogue/speech (with precise transcription), music, ambient sound/atmosphere (ambient sounds, background), and foley/sound effects. The paper emphasizes that LTX-2 goes "beyond merely generating speech," instead producing a complete soundscape that follows the character, environment, style, and emotional tone.
【Undisclosed】the respective proportions of the four audio categories, the retained proportion of silent clips, the method for controlling category mixture, and the specific quantitative criteria for "significant and informative" (no SNR/loudness/silence-ratio thresholds are given). The model card also acknowledges that "audio quality is lower when generating non-speech content," suggesting that speech-category data likely makes up a significantly larger share than sound-effect/music-category data. [Uncertain]

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track) ⚠️

Neither the single-shot vs. multi-shot mixture nor the average shot-count distribution is disclosed. What can be confirmed is that the data-organization granularity is single-shot: the pipeline runs from "Input Shots" to "Final Shots" entirely at shot granularity, so training samples are predominantly single-shot clips, and the model does not market multi-shot narrative capability (the paper explicitly lists "deeper narrative coherence" as a limitation that must be compensated for by relying on an external LLM to generate the conditioning text). Presence of a native audio track: by definition, all samples in the LTX-2 subset come with their own native, audio-informative synchronized audio tracks — this is precisely what distinguishes it from the parent dataset. For average clip duration, refer to LTX-Video's duration histogram (0–30 seconds). [Uncertain]

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

No list of languages or their proportions is given, but language/accent is a first-class citizen on both the annotation side and the architecture side.
【Annotation side】The captioning system applies "precise transcription plus speaker, language, and accent identification" to dialogue — this is one of the most valuable labeling designs in this entry, directly forming the data foundation for multilingual lip sync and controllable accents.
【Architecture side】Gemma3-12B, a multilingual LLM, is used as the text encoder, with multi-layer feature extraction (early layers carrying raw phonemic information, later layers carrying complex semantics); the team explicitly states that "deep textual understanding serves not only global language support but also directly determines the phonemic accuracy of the generated speech." At inference time, users can enclose dialogue lines in quotation marks within the prompt and specify the desired language and accent.
【Limitation】The paper acknowledges that for languages or dialects underrepresented in the training data, speech-synthesis accuracy and audio-visual alignment degrade noticeably. Exactly how many languages are covered, and the proportion of samples per language/accent, is entirely undisclosed. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of levels, ordering)

LTX-2 itself does not present a new cleaning-funnel diagram; its video side entirely reuses the LTX-Video pipeline, with an additional layer of "audio-informativeness filtering" stacked on top of its output.
【LTX-Video data-processing pipeline (paper Fig.11, 9 levels total, in the following order)】
1. Input Shots (shots as the input unit)
2. Crop Black Bars (remove black borders, standardize aspect ratio)
3. Estimate Motion Level (estimate motion magnitude)
4. Generate Thumbnails
5. Mid-Frame CLIP Image Embedding (compute CLIP image embeddings from the middle frame)
6. Cluster and De-Duplicate
7. Resize Shots (size normalization)
8. Estimate Aesthetics (aesthetic scoring)
9. Filter Shots → Final Shots (filter by thresholds on motion magnitude/aesthetic score, etc., producing the final shot set)
This is followed by an independent "Captioning and Metadata Enhancement" stage — the internal automatic captioner re-captions the entire training set.
【LTX-2 increment】On top of Final Shots, an AV subset is extracted using "contains significant and informative audio components" as the criterion, and re-captioned with a newly developed dual-track audio-video captioner.
【Structural characteristics】This is a typical five-stage structure of "geometric/structural normalization first → representation extraction → deduplication → quality-score filtering → captioning last," dominated by shallow discriminators (CLIP embeddings + an aesthetic Siamese network + motion estimation), with no VLM/LLM semantic quality-inspection step observed.

### Quantitative funnel retention rate (input/output volume at each level and final retention rate, e.g. Apollo's 27%) ⚠️

Entirely undisclosed. Neither the LTX-2 nor the LTX-Video paper gives the input/output sample count, elimination rate, or final retention rate for any filtering level, so no comparison is possible against quantitative funnels like Apollo's 27%. It is known only that the LTX-2 training set is "a subset" of the LTX-Video dataset, but the proportion the subset represents of the parent set is likewise unpublished. [Uncertain]

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

The pipeline's input unit is directly labeled "Input Shots," indicating that shot splitting has already been completed upstream, and the entire pipeline (motion estimation, deduplication, resizing, aesthetic scoring, filtering) operates on shots as the atomic unit, with "Final Shots" as the final product. However, neither paper states the specific method used for shot splitting — neither mentioning open-source tools such as PySceneDetect/TransNetV2, nor stating whether an in-house shot-aware splitting model is used, and no threshold parameters are given. [Uncertain]

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

Quality filtering on the video side is the most finely disclosed part of the LTX series, centered on an in-house pairwise-preference aesthetic scoring model:
(1) Labeling-pair sampling: a multi-labeling network first tags millions of samples, and pairs are sampled only from samples that "share at least one of their top-3 labels" — the goal being to ensure comparisons occur between similar content, minimizing the distribution shift introduced by aesthetic filtering. This design is the single most worth-emulating detail in the pipeline.
(2) Human annotation: tens of thousands of image pairs are labeled by humans indicating which image is more aesthetically superior.
(3) Model training: these preference pairs are used to train a Siamese Network that learns an aesthetic score preserving the labeled ordinal relationships (a ranking-based rather than absolute-regression score), capable of evaluating both video and images.
(4) Application: an aesthetic score is computed for every sample, and those below a threshold are removed (the threshold value is not published); the fine-tuning stage further retains only the subset with the highest aesthetic scores.
(5) Geometric cleaning: explicit black-bar cropping ("Crop Black Bars") removes black borders.
【Not mentioned】No description whatsoever of OCR/subtitle text filtering, watermark/logo detection, sharpness/blur criteria, or compression-artifact detection. [Uncertain]

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content) ⚠️

Level 3 of the pipeline is "Estimate Motion Level," used as the basis for filtering in the Filter Shots step. The paper states explicitly: "we actively remove videos with insignificant motion to ensure the dataset focuses on dynamic content," reasoning that dynamic content better matches the model's target capabilities. Neither the specific algorithm used to estimate motion magnitude (whether optical flow, frame differencing, or something else) nor the threshold value is stated, and there is no mention of whether jittery/handheld-shake motion is separately removed. [Uncertain]

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

A representation-based semantic deduplication approach is used: for each shot, a CLIP image embedding is computed from the middle frame, and "Cluster and De-Duplicate" is then performed in embedding space. This is embedding-level semantic deduplication rather than hash-level exact deduplication; the paper does not separately state whether an additional exact-deduplication step exists, nor does it disclose the clustering algorithm, similarity threshold, within-cluster retention policy, or the deduplication ratio. [Uncertain]

### VLM/LLM as quality inspector (multimodal-large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

The LTX series does not use a VLM/LLM as a data quality inspector — a notable point of divergence from the mainstream 2026 trend. All judgment roles in the data pipeline are filled by shallow/dedicated models: a CLIP image encoder (for deduplication representations), a multi-label classification network (for pairing samples), a Siamese aesthetic-ranking network (for quality scoring), and a motion-magnitude estimator. Large models play a role in LTX-2 in only two places, and neither is quality inspection: (1) the in-house captioner (dual-track video+audio labeling); (2) Gemma3-12B as a frozen text encoder participating in conditional modeling. The technical report makes no mention of using a multimodal large model for semantic quality scoring or for removing image-text/audio-visual mismatches. Whether this is used in an undisclosed internal process cannot be determined. [Uncertain]

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

Safety and compliance filtering on the training-data side is entirely undisclosed: there is no description whatsoever of NSFW filtering, face/privacy handling, or CSAM screening. The team's compliance strategy is primarily front-loaded at the data-acquisition stage — ensuring copyright cleanliness through the Shutterstock research license and Getty Images licensed acquisition, rather than through downstream content classifiers. The technical report's "Social Impact" section only makes qualitative statements at the deployment level: acknowledging the risk that synthetic media could be used for deceptive content, requiring users to explicitly disclose synthetic origin and follow content-safety guidelines, acknowledging that the model inherits visual and auditory biases present in the training data, and listing bias mitigation, authenticity verification, and provenance as future work. The model card's Limitations section explicitly notes that the model "may generate inappropriate content, amplify societal biases." No description of any automated safety-filtering module, classifier, or red-teaming process is found. [Uncertain]

## Captioning Approach

### Caption models used (in-house VLM/open-source model, model scale) ⚠️

An in-house internal automatic captioner developed by Lightricks is used; it is not open-sourced, and neither its model scale nor its base model is disclosed.
【Predecessor LTX-Video】Used an "internal automatic image and video captioner" to re-caption the entire training set, ensuring that text descriptions are accurate and relevant and improving the alignment between visual content and text annotation.
【LTX-2 increment】Section 5.1 of the paper states explicitly: "a new video captioning system" was developed specifically for training LTX-2, capable of describing both the visual and auditory tracks of a clip in exhaustive detail, "capturing every meaningful action, appearance, and sound." This system is positioned as "a comprehensive textual interface bridging video, audio, and language, forming the descriptive foundation of LTX-2's multimodal training corpus."
【Undisclosed】the captioner's parameter count, base model, whether it is a multimodal LLM, how audio understanding is implemented (whether via a combined pipeline of ASR + audio event detection + speaker diarization), and the labeling throughput and cost. Note that Gemma3-12B is the text encoder used during training/inference, not the captioner. [Uncertain]

### Caption density and degree of structuring (short/long/dense description, structured fields such as camera motion, style tags)

Follows a route of "extremely long, dense, single coherent paragraph, purely factual" — a typical dense-caption paradigm.
【Density】LTX-Video's paper Fig.14(a) gives the word-count distribution per caption, spanning roughly 0–175 words, concentrated mainly in the tens of words; LTX-2's captions are longer and denser still, since they must simultaneously cover the auditory track. The official prompting guide recommends users write 4–8 descriptive sentences, kept as a single coherent paragraph — the inference-prompt format mirrors the training-caption format.
【Style constraint】The paper explicitly states the labeling standard: "comprehensive yet factual, describing only what is seen and heard without emotional interpretation" — comprehensive but strictly factual, describing only what is seen and heard, with no emotional interpretation or subjective evaluation. This is a very clearly stated caption style guideline.
【Degree of structuring】No explicit JSON/key-value fields are used; instead, structured information is fused into a long, natural-language description organized chronologically. LTX-Video's example captions show a fixed set of semantic slots covered: subject appearance and detail → subject action/behavior → scene and background elements → camera position and movement (e.g., "camera follows … from behind, pans slightly" / "camera remains stationary") → lighting and color ("natural and slightly overcast lighting, casting soft shadows") → style/source tags (a fixed suffix such as "The scene is captured in real-life footage." or "The scene appears to be from a movie or TV show."). LTX-2 builds on this by adding auditory slots.

### Joint audio-video caption structure (whether both visual and auditory tracks are covered, whether split into independent fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

This is the core contribution of this entry, and among currently available public materials, one of the best examples of joint audio-video caption design.
【Design principle】A single caption exhaustively covers both the visual and auditory tracks of a clip in exhaustive detail, rather than being split into multiple independent fields — a different route from Script-a-Video's factorized streams or Foley-Omni's three fields: LTX-2 takes the "fused, all-information long-description" route.
【Auditory-side coverage (full soundscape)】
- Music: genre, instrumentation, tempo, mood
- Ambient sound/atmosphere (ambient sounds, background): wind, rain, urban background noise, indoor room tone, crowd chatter
- Foley/sound effects: footsteps, clothing rustle, keyboard clacking, door creaks, glass clinks
- Dialogue: precise transcriptions, together with speaker, language, and accent identification
【Visual-side coverage】Camera motion, lighting, subject behavior, as well as appearance, scene, and style/source.
【Consistency design】The official prompting guide is isomorphic to the training-caption structure — user prompts are likewise required to be a single paragraph, chronologically ordered, explicitly describing ambience/foley/music/speech, with dialogue lines placed in quotation marks and optionally annotated with language and accent, and using physical cues rather than emotional labels to direct performance. This confirms the design closed-loop of "the training caption is the inference interface."
【Significance】Audio is no longer subordinate to the visual caption but is a descriptive dimension on equal footing with the visual one; "audio-informativeness filtering (data side) + full-soundscape dual-track captioning (annotation side)" together form the complete logic of LTX-2's data methodology: first ensure the sample's audio track carries information, then ensure the annotation fully exposes that information to the model.
【Undisclosed】the length ratio between the visual and auditory portions within a caption, any template or slot definition for the auditory description, and whether there is any quality-verification step.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion)

Explicitly performed, and an explicit component of the caption schema: captions include "precise transcriptions of dialogue," together with annotation of three speaker attributes — speaker identity, language, and accent. These three attribute annotations directly underpin the model's multilingual lip-sync, speaker differentiation, and controllable-accent capabilities (the paper states the model can synthesize speech that is "not only synchronized with lip movement but also natural in rhythm, accent, and emotional tone").
A notable contrast: emotion is not used as an annotation field — the labeling standard explicitly requires "no emotional interpretation," describing objectively only what is seen and heard; emotional expression is expected to be learned indirectly through dialogue content, physical-action cues, and timbre description. The official prompting guide recommends that users break dialogue lines into short sentences, insert acting directions between sentences, and use physical cues rather than emotional labels.
Undisclosed: the specific ASR model used, the speaker-separation/diarization method, the language- and accent-identification model, transcription word-error rate, or quality-control measures. In multi-speaker scenes, the model still exhibits dialogue lines misattributed to the wrong character (a paper-stated limitation), suggesting that speaker-to-dialogue binding, whether in annotation or modeling, is still not fully robust.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

Essentially none. Neither paper mentions camera intrinsics/extrinsics, depth maps, 3D point tracks, optical-flow fields, explicit physical state, or action-category labels as structured geometric annotation. The only related item is natural-language-level camera description — the caption records camera movement in text form (tracking shots, pans, static shots, etc.), which counts as weakly structured. An additional clue: officially released camera-motion control LoRAs and pose control LoRAs indicate the existence of purpose-built control datasets, but their construction method (whether explicit camera trajectories or pose-estimation annotations are used) is not disclosed. [Uncertain]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV) ⚠️

The technical report makes no mention of using any synthetic data or controlled perturbation/editing to construct training pairs (no InstructAV2AV-style edited pair construction). All training data comes from real video (publicly available + licensed material) plus image datasets. The only place involving "model-generated content" is in the LTX-Video paper, where Sora-generated video is used as a demonstration sample of the captioner's effectiveness — a demonstration, not training data. Whether distilled/self-generated data is used in an undisclosed process (such as for training the distilled model) cannot be determined. [Uncertain]

### Degree of human involvement (human annotation, human quality inspection, model-prescreen + human review)

Human involvement is concentrated in building the quality-judgment models, not in per-sample annotation:
(1) Aesthetic-model labeling: tens of thousands of image pairs are labeled by humans indicating "which image is more aesthetically superior," used to train the Siamese ranking network; upstream, a multi-label network first auto-tags millions of samples to constrain the pairing range. This is a classic leveraged design of "human-label a small number of preference pairs → train an automatic scorer → filter the full volume automatically."
(2) Per-sample captioning: fully automatic, with the internal captioner re-captioning the entire training set; no human review or spot-check process is mentioned.
(3) Evaluation: a human preference study is used to assess visual realism, audio fidelity, and temporal synchronization (lip sync, foley accuracy); the LTX-Video-stage human evaluation used 1,000 text-to-video prompts and 1,000 image-to-video sample pairs.
Whether there is a human fallback quality-inspection step in the data-cleaning process is not disclosed.

## Audio-Video Alignment

### Audio-video synchronization detection methods (lip sync, event alignment) ⚠️

This is the single biggest gap in LTX-2's data disclosure, and also the piece most relevant to — yet most lacking for — this survey's topic. The technical report describes no audio-video synchronization detection or asynchronous-sample-removal mechanism whatsoever on the training-data side — no lip-sync detector, no event-alignment detection, no asynchrony-removal threshold.
The source of its synchronization capability is attributed to three factors, all on the data-selection and architecture side rather than the sync-detection side:
(1) Data side: only clips with "significant and informative audio components" are retained — i.e., real videos with native, self-contained synchronized audio tracks are kept, which naturally avoids asynchronous samples such as dubbed or post-added audio (though the paper does not state whether voice-overs, post-production scoring, or other non-synchronous sound is explicitly excluded);
(2) Annotation side: captions precisely transcribe dialogue and annotate speakers, allowing text-speech-lip movement to be aligned at the semantic level;
(3) Architecture side: bidirectional audio-video cross-attention runs throughout the full depth, and cross-modal attention uses only 1D temporal RoPE, forcing attention to concentrate on the temporal-synchronization dimension; the paper claims this maps visual events (e.g., an object impact) to auditory events (the corresponding foley sound) with "sub-frame precision"; cross-modality AdaLN further regulates the strength of cross-modal information injection; increasing the cross-modal guidance s_m at inference time improves temporal synchronization.
Paper Fig.3 offers visualizations of the AV cross-attention maps as qualitative evidence of synchronization capability: the model can spatially track a moving vehicle, dynamically switch attention between two speakers, and focus on the lip region during close-up speech. [Uncertain]

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 ∧ LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

Entirely undisclosed. The paper uses no SyncNet, AV-align, or any in-house synchronization metric, gives no confidence-threshold values (so no comparison is possible against, e.g., UniTalking's SyncNet conf>0.9), and reports no score on any objective audio-video synchronization benchmark for LTX-2. All audio-video quality conclusions come from an internal human preference study (comparing against Ovi, Veo 3, Sora 2), and the paper does not give a specific score table for the human evaluation, describing results only qualitatively as "significantly better than Ovi" and "comparable to leading closed-source models." The only quantitative table is an inference-speed comparison (on an H100, 121 frames at 720p, single-step Euler, CFG=1: LTX-2 19B audio-video at 1.22 seconds/step vs. Wan 2.2-14B video-only at 22.30 seconds/step, roughly an 18× speedup). [Uncertain]

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching treated as two independent filtering conditions) ⚠️

The paper does not distinguish "temporal synchronization" from "semantic matching" as two independent conditions at the data-filtering level — on the data side, there is only the single filtering criterion of "audio informativeness." However, in the methodological narrative and architectural design, the two are clearly distinguished and mapped to separate mechanisms:
(1) Temporal synchronization: handled by retaining only the temporal component of RoPE in cross-modal attention (explicitly described as "forcing cross-modal attention to focus on temporal synchronization rather than spatial alignment"), targeting lip sync and sub-frame impact-to-sound-effect alignment;
(2) Semantic/environmental matching: handled by bidirectional dependency modeling — the paper's central argument is that "lip sync is primarily driven by audio, while the acoustic environment (reverb, foley) is determined by visual context," which is precisely why bidirectional dependencies must be jointly modeled, and is the main argument against cascaded V2A/A2V approaches; increasing the cross-modal guidance s_m at inference time improves both "temporal synchronization" and "semantic coherence" simultaneously, and the paper presents the two side by side.
Whether corresponding filtering conditions of these two types exist on the data side is not disclosed. [Uncertain]

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, removal of no-audio-track clips, removal of off-screen sound sources, background-music separation) ⚠️

The single core audio-side filtering criterion is "significant and informative audio components" — this effectively serves simultaneously as the mechanism for removing clips with no audio track, removing silent/near-silent clips, and removing low-informativeness audio tracks (e.g., pure noise, extremely low loudness); the paper describes it as the key factor giving the subset a "balanced distribution of visual and auditory content."
【Undisclosed】the quantitative implementation of this criterion: no SNR threshold, no loudness/RMS threshold, no silence-ratio threshold, no audio-event-density metric; it is not stated whether off-screen sound sources such as voice-over/narration are removed, nor whether background-music separation (source separation) or vocal-accompaniment separation is performed. This is a typical case of "the idea is clear but the implementation details are entirely non-reproducible." On the technical side, all that is known is that audio enters the VAE as a 16kHz stereo mel spectrogram and is output at 24kHz; requirements on the sampling rate/channel count of the raw admitted audio track are not stated. [Uncertain]

### Classification and separate handling strategies for speech/sound effects/music ⚠️

At the caption-annotation level, the four audio categories are explicitly distinguished and described separately — speech (precise transcription + speaker/language/accent), music, ambient sound/atmosphere (ambient, background), and foley/sound effects — and this is the foundation enabling the model to respond separately to instructions for each of the four audio categories.
However, at the data-filtering and training-mixture level, the paper does not state whether separate filtering rules, mixture targets, or loss weights are designed for each of the four categories — "audio-informativeness filtering" is a single unified gate, with no observed category-specific routing. Architecturally, audio is modeled uniformly as a single 5B stream, with no category-specific branching or expert routing.
Indirect evidence points to category imbalance: the model card's Limitations section explicitly states that "the model may produce lower-quality audio when generating non-speech content," indicating that speech-category data dominates the subset, with sound-effect/music-category data relatively underrepresented. The paper also acknowledges that deep textual understanding mainly serves the phonemic accuracy of speech, with the emphasis clearly skewed toward speech. [Uncertain]

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

The training curriculum is partially disclosed, but lacks detail on the data-dimension stage division.
(1) Modality progression: the core idea is "extending a pretrained video DiT with an audio stream" — the conclusion section states this as "extending a pretrained 13B video diffusion transformer with a lightweight 3B audio stream" (the specification given in the body and abstract is 14B video + 5B audio, so there is a discrepancy in the figures cited), connected via bidirectional cross-attention, 1D temporal RoPE, and cross-modality AdaLN, so that the visual backbone need not be duplicated; this is followed by "progressive joint training." This is a clearly stated "video first, then audio-video" modality curriculum.
(2) Two-stage text projection: the projection matrix W for multi-layer feature extraction is jointly optimized with LTX-2 during a brief initial training stage (the LLM's weights are frozen throughout, using a standard diffusion MSE loss), and this stage brings an overall quality improvement; afterward, W is frozen and reused for all subsequent training. The text connector (including thinking tokens) is trained together with the audio-video DiT blocks.
(3) Resolution/duration: rather than a strict low-res→high-res staged progression, training occurs simultaneously across multiple resolutions and durations combined with token-count alignment and stochastic token dropping; high resolution is achieved through a multi-scale, multi-tile inference-time strategy (0.5MP base → latent upsampling → tile refinement) rather than through a training curriculum.
(4) Image participation: images participate in mixed training as one "resolution-duration combination."
(5) Predecessor LTX-Video: after pretraining, fine-tuned on a high-aesthetic subset.
The step counts, data volumes, and switching criteria for each stage are not disclosed. [Uncertain]

### How data mixture changes across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

LTX-2 does not disclose how the data mixture changes across training stages, gives no description of an annealing stage, and does not state whether purely visual (no-audio-track) data is still mixed in during the joint audio-video training stage to preserve visual quality — given that the paper emphasizes that "joint training does not degrade single-modality visual quality" (ranking 3rd in I2V and 4th in T2V on the Artificial Analysis leaderboard, surpassing Sora 2 Pro and Wan 2.2-14B), one might speculate that some mixing of purely visual data, or partial protection of the video-stream parameters, exists — but there is no official basis for this. The only comparable disclosed practice is LTX-Video's "pretrain on the full volume → fine-tune using only the highest-aesthetic subset." [Uncertain]

### Post-training data (scale and selection criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

Entirely undisclosed. The technical report has no SFT/RLHF/DPO section, gives no curated-set scale or selection criteria, no number of preference pairs, and no description of reward-model training data. Known post-training artifacts whose data provenance is not stated include: the 8-step distilled version ltx-2-19b-distilled (the distillation-data construction method is not disclosed), fp8/fp4 quantized versions (calibration data not stated), and a series of control LoRAs (camera motion, pose control, lip dubbing) — the latter obviously require purpose-built paired datasets, but the construction method is not disclosed. The human preference study is used only for final evaluation, and it is not stated whether its results feed back as a training signal. [Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Entirely undisclosed. The technical report makes no mention of NeMo Curator, Data-Juicer, or any in-house data-processing framework, and gives no GPU acceleration ratio, processing throughput, processing scale, or cost figures, nor does it describe how the data pipeline is distributed. All efficiency figures in the paper pertain to the inference side: on an H100, 121 frames at 720p takes 1.22 seconds per step (compared to Wan 2.2-14B's 22.30 seconds, roughly 18×), and the paper emphasizes that this gap widens further at higher resolutions and longer durations; one of the model's design goals is to be runnable locally on consumer-grade GPUs. [Uncertain]

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

No data-strategy ablation experiments exist at all. The technical report has no filtering-strictness ablation, no caption-density/style ablation, no data-mixture ablation, and no comparison of "with audio-informativeness filtering vs. without."
All existing ablations/comparisons are concentrated on the architecture and inference side:
(1) Modality-CFG: an effect analysis independently tuning s_t (text guidance) and s_m (cross-modal guidance) is given (Fig.5), concluding that increasing s_m improves temporal synchronization and cross-modal semantic consistency, with the final settings being (3,3) for the video stream and (7,3) for the audio stream;
(2) Text-conditioning design: qualitatively states that multi-layer feature extraction outperforms using only the final-layer causal embedding, and that thinking tokens significantly improve speech phonemic accuracy and adherence to complex prompts, but no quantitative comparison table is given; the initial joint training of the projection matrix W is said to "bring an overall quality improvement," likewise with no figures;
(3) Efficiency comparison: the single-step-time table against Wan 2.2-14B (the only quantitative table);
(4) Quality comparison: an internal human preference study (vs. Ovi / Veo 3 / Sora 2) and the public Artificial Analysis leaderboard (as of 2025-11-06, ranking 3rd in I2V and 4th in T2V), neither with score breakdowns. [Uncertain]

### Evidence on quality vs. quantity (cases where small, precise data beats large, messy data) ⚠️

Implicit evidence exists but with no controlled experiment to back it. LTX-2's entire data strategy is precisely "filtering an audio-informativeness-significant subset out of the larger LTX-Video dataset" — training an open-source SOTA joint audio-video model on a true subset of the parent set, without any degradation in visual quality (still surpassing Sora 2 Pro and Wan 2.2-14B on the leaderboard) — which is itself a case of "targeted filtering outperforming brute-force accumulation of volume." Similarly, LTX-Video's fine-tuning stage uses only the highest-aesthetic-score subset, reflecting the same orientation.
But neither paper conducts a "full volume vs. subset" controlled experiment, gives the subset's proportion of the whole, or quantifies the benefit of "quality over quantity." So this can only be counted as a methodological inclination, not experimental evidence. [Uncertain]

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g. VABench's seven major categories) ⚠️

The domain taxonomy of the training data is not disclosed, so no statement can be made about its alignment with any evaluation-benchmark taxonomy (no categorized benchmark such as VABench's seven categories is involved). LTX-2's evaluation strategy is "human preference + public leaderboard" rather than a categorized benchmark: (1) an internal human preference study, evaluated along three dimensions — visual realism, audio fidelity, and temporal synchronization (lip sync and foley accuracy) — comparing against Ovi, Veo 3, and Sora 2; (2) the I2V and T2V tracks of the public Artificial Analysis leaderboard. VBench is not used, no objective audio-video synchronization metric is used, and no proprietary categorized evaluation set is constructed or published. The evaluation dimensions (visual/audio/synchronization) correspond conceptually, one-to-one, with the data-construction logic (visual-quality filtering / audio-informativeness filtering / joint captioning), but this is a structural coincidence rather than an alignment design stated in the paper. [Uncertain]

## Other Information

### summary_note

【Positioning】LTX-2 is the most representative example of the gap, currently, between "degree of openness" and "legibility of data methodology": weights, inference code, and training code are all fully open (the most thorough among peers), yet the "Training Data" section of the 14-page technical report is only two paragraphs of about 150 words, with not a single data statistic. Its data-side reproducibility is far lower than its model-side reproducibility.
【The three most valuable takeaways】
1. Audio-informativeness filtering: rather than directly repurposing a purely video dataset for audio-video training, a subset is extracted using "whether the audio component is significant and informative" as the criterion, accomplishing in one step the removal of silent/near-silent/low-information audio tracks and achieving a balanced distribution of audio-visual content. This is the first-principles data operation that distinguishes AV models from purely visual models — the idea is clear and transferable, but there is no quantitative criterion given.
2. Dual-track full-soundscape captioning (the core highlight of this entry): a single caption exhaustively covers both the visual track (camera motion, lighting, subject behavior, appearance, scene, style/source) and the auditory track (music, ambient sound, foley, precise dialogue transcription + speaker/language/accent annotation), with an explicit style guideline of "comprehensive yet factual, describing only what is seen and heard without emotional interpretation." Audio is elevated to a descriptive dimension on equal footing with vision, and the training-caption structure is isomorphic with the inference-prompt structure (the official prompting guide requires a single paragraph, chronological order, explicit description of ambience/foley/music/speech, dialogue lines in quotation marks annotated with language and accent), forming a complete data-interface closed loop. The three-attribute annotation of speaker/language/accent is the direct data foundation for the multilingual lip-sync capability.
3. A reusable video-cleaning funnel (inherited from LTX-Video): a 9-level shot-granularity pipeline (black-bar cropping → motion-magnitude estimation → thumbnail generation → mid-frame CLIP embedding → cluster-based deduplication → resizing → aesthetic scoring → threshold filtering), whose aesthetic-scorer construction method is the most worth emulating — first using a multi-label network to constrain the pairing range (pairing only samples sharing at least one of the top-3 labels) to minimize the distribution shift introduced by filtering, then training a Siamese ranking network on tens of thousands of human preference pairs, and finally applying it as an automatic scorer/filter at full scale.
【The biggest information gap】Audio-video synchronization detection: for a model whose main selling point is "temporal synchronization," its data pipeline discloses no lip-sync/event-alignment detection step whatsoever — no SyncNet-type metric, no threshold, no description of asynchronous-sample removal. Synchronization capability is attributed entirely to "native synchronized audio-track data + bidirectional cross-attention architecture + temporal-dimension RoPE + modality-CFG," and whether any data-side synchronization quality control exists cannot be determined. The second-biggest gap is the absence of essentially all quantitative information: data scale, funnel retention rate, audio-category mixture, language distribution, caption-length distribution (only the predecessor has a figure, with no numbers), and data ablations — all are zero.
【Compliance profile】The route taken is "front-loaded licensed data" in place of "downstream safety filtering": the Shutterstock research license (December 2024, first global partner) + the Getty Images strategic partnership position the model as commercially safe; but the original paper text states "publicly available data supplemented with licensed material," which diverges from the media framing of "entirely licensed" — the proportion of licensed data is unpublished, and there is no output-side provenance mechanism such as C2PA.
【Comparison with peers】Compared to Sora 2 (near-zero data-side disclosure) and Veo 3, LTX-2 at least provides qualitative design descriptions of its filtering criteria and caption schema, and has runnable open-source code as corroboration; compared to Ovi (a symmetric dual-backbone with two 5B streams), LTX-2's asymmetric dual-stream design is more efficient. For anyone seeking a reproducible AV data-processing method, what LTX-2 offers is a "conceptual template," not a "parameter recipe."

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

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
- geometric_structured_annotation
- synthetic_data_synthesis
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
