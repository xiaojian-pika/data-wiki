# Sora 2 (including Sora 2 Pro)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

Sora 2 (including Sora 2 Pro)

### Publishing Organization/Company

OpenAI

### Release Date (technical report/paper/open-source date)

Model and System Card released on September 30, 2025 (sora.com and a standalone iOS Sora app went live the same day); the API (sora-2 / sora-2-pro) opened in early October 2025, and the duration cap was extended from 10 seconds to 15 seconds (25 seconds on the Pro web version); a three-year licensing agreement with Disney was reached in December 2025; in March 2026, OpenAI announced the shutdown of the consumer-facing Sora app (the app went offline on April 26, 2026, and the API was discontinued on September 24, 2026), and Disney's $1 billion investment and licensing agreement were terminated as a result. Note: OpenAI never published a technical report or paper for Sora 2 — only a 7-page System Card.

### Type (model/dataset/toolchain/evaluation benchmark)

A closed-source commercial model (a native joint audio-video generation video foundation model + a consumer social app + an API service). Not a dataset, not a toolchain, not an evaluation benchmark.

### Openness (whether weights/code/data/pipeline are each open source)

Fully closed. Weights are not open, code is not open, training data is not open, and the data-processing pipeline is not open. The only public material is the "Sora 2 System Card" from September 30, 2025 (7 pages total), in which content about data is limited to a single paragraph (about 5 sentences) in Section 2, "Model Data & Data Filtering." There is no technical report, no paper, no architectural details, and no data statistics whatsoever. By comparison, the prior generation Sora 1 at least had a technical blog post, "Video generation models as world simulators," which disclosed methodology such as spacetime patches, native-resolution training, and re-captioning. Sora 2 was once commercially available via API (sora-2, sora-2-pro), and by September 2026 the API had also been discontinued.

### Whether it supports simultaneous audio-video generation, and the implementation approach (native joint / cascaded / MoE fusion) ⚠️

Yes, and as native joint generation — this is the core upgrade of Sora 2 relative to Sora 1. The System Card explicitly positions it as a "video and audio generation model," with new capabilities including "synchronized audio." Audio is not a post-hoc cascaded video-to-audio module, but is jointly denoised together with video within the same generation pipeline: video and audio are each compressed into latents via their own encoders, and then a single shared transformer diffusion backbone denoises both latent streams simultaneously. It can generate dialogue (including lip sync), sound effects/foley, ambient sound, and background music, with volume and spatial positioning varying with the distance of objects and camera. Note: the above architectural description (dual encoders + shared diffusion backbone, 3D RoPE, an audio backbone sharing lineage with the GPT-4o multi-modal system) all comes from third-party technical analyses and secondhand reporting; OpenAI has never officially confirmed this, so it should be treated as speculative information. [Uncertain]

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature noted: primary official / same-team corroboration / third-party report)

- Sora 2 System Card, OpenAI, 2025-09-30 (PDF): https://cdn.openai.com/pdf/50d5973c-c4ff-4c2d-986f-c72b5d0ff069/sora_2_system_card.pdf
- Sora 2 System Card index page: https://openai.com/index/sora-2-system-card/
- OpenAI Deployment Safety Hub - Sora 2: https://deploymentsafety.openai.com/sora-2
- Sora System Card (Sora 1, includes CSAM safety-stack details): https://openai.com/index/sora-system-card/
- Video generation models as world simulators (Sora 1 technical blog, covers spacetime patches/native resolution/re-captioning): https://openai.com/index/video-generation-models-as-world-simulators/
- Sora 2 is here (launch announcement): https://openai.com/index/sora-2/
- Disney and OpenAI licensing agreement announcement: https://openai.com/index/disney-sora-agreement/
- Sora, Not Sorry: OpenAI Backtracks on Opt-Out Copyright Policy, Copyright Lately: https://copyrightlately.com/openai-backtracks-sora-opt-out-copyright-policy/
- Sora 2 Does A Copyright Somersault Upon Launch, Forbes, 2025-10-17: https://www.forbes.com/sites/legalentertainment/2025/10/17/sora-2-does-a-copyright-somersault-upon-launch/
- MPA calls on Sora 2 to stop infringement, CNBC, 2025-10-07: https://www.cnbc.com/2025/10/07/openais-sora-2-must-stop-allowing-copyright-infringement-mpa-says.html
- Japanese government requests OpenAI address Sora 2 infringement, EC IP Helpdesk: https://intellectual-property-helpdesk.ec.europa.eu/news-events/news/japanese-government-requests-openai-avoid-copyright-infringement-sora-2-us-federal-judge-dismisses-2025-10-23_en
- Public Citizen open letter calling on OpenAI to withdraw Sora 2: https://www.citizen.org/news/public-citizen-letter-calls-on-open-ai-to-withdraw-sora-2-video-generation/
- OpenAI Will Shut Down Sora Video App; Disney Drops $1B Investment, Variety, 2026-03: https://variety.com/2026/digital/news/openai-shutting-down-sora-video-disney-1236698277/
- Sora Shutting Down, Disney Investment Dead, Deadline, 2026-03: https://deadline.com/2026/03/sora-shut-down-disney-investment-1236764689/
- OpenAI Adds Longer Clips and Storyboarding to Sora 2, eWeek: https://www.eweek.com/openai/openai-adds-longer-clips-sora-2/
- How OpenAI Built Sora 2: Training, Data, and Model Design (third-party technical analysis, unofficial): https://skywork.ai/blog/openai-sora-2-2025-ultimate-guide-training-model-design/
- How to Uncover Sora 2's Training Datasets (third-party, unofficial): https://skywork.ai/blog/how-to-uncover-sora-2s-training-datasets/
- Sora 2 API on Replicate (specs and pricing): https://replicate.com/openai/sora-2
- Getty Images/OpenAI licensing partnership report (2026-06): https://finance.yahoo.com/markets/stocks/articles/getty-images-openai-deal-gives-154500732.html

## Data Scale and Distribution

### Training data volume (number of videos/hours/tokens, pretraining and SFT separated) ⚠️

Completely undisclosed. The System Card gives no video count, total hours, token/patch count, no breakdown between pretraining and SFT scale, and no disclosure of compute budget. OpenAI has never given any official data-scale figures externally either. [Uncertain]

### Data source composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

Only one extremely high-level qualitative description exists. Original System Card text: "Sora 2 was trained on diverse datasets, including information that is publicly available on the internet, information that we partner with third parties to access, and information that our users or human trainers and researchers provide or generate." That is, three source types: (1) publicly available internet data (web crawling); (2) data obtained through third-party partnerships/licensing; (3) data provided or generated by users, human trainers, and researchers. No proportion is given for any source, no specific dataset names, no scope of crawling, and no list of partners. Whether synthetic data is used is not clearly stated (the word "generate" may hint at including content generated by human trainers, but this is not the same as model-synthesized data). [Uncertain]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.)

Disclosure on the training side of provenance is extremely weak, while disclosure on the output side of provenance is comparatively strong; the two must be strictly distinguished.
[Training side] The proportion of licensed data is not disclosed, no list of rights-cleared datasets is published, and no procurement parties are named. The only clear compliance statement concerns child safety: "responsibly sourcing datasets to exclude CSAM" (responsibly filtering data sources to exclude child sexual abuse material), in partnership with the National Center for Missing & Exploited Children (NCMEC) in the US. On copyright: at launch, Sora 2 adopted an "opt-out" policy (rights holders had to proactively request exclusion), which triggered controversy over mass generation of IPs such as SpongeBob, South Park, and Scooby-Doo; on October 3, 2025 (just 3 days after launch), Sam Altman published a blog post switching to "opt-in" and promised rights holders finer-grained control and revenue sharing. The Motion Picture Association (MPA) publicly pressured OpenAI, and the Japanese government also formally requested that OpenAI avoid infringement. Key point: the opt-in policy only constrains the "generation" stage — OpenAI has never clarified whether this policy is applied retroactively to "training data," meaning copyrighted content already trained into the model was not removed. OpenAI subsequently reached licensing agreements with Disney (December 2025, a three-year license covering 200+ Disney/Marvel/Pixar/Star Wars characters) and Getty Images (June 2026, a multi-year partnership following Getty's $3.7 billion merger with Shutterstock), but these are all "generation-side IP licensing/display partnerships," not explicitly licensing agreements for Sora 2's training corpus.
[Output side] All first-party product assets carry C2PA metadata (an industry-standard verifiable provenance mechanism); videos downloaded from sora.com and the Sora App carry a visible moving watermark; OpenAI maintains internal detection tools to determine whether a given video/audio was generated by its products. OpenAI itself acknowledges that "there is no single solution" to provenance.

### Clip duration distribution and segmentation strategy ⚠️

The training data's clip duration distribution and segmentation strategy are completely undisclosed. Only inference-side output specs are known: initially 10 seconds; after the October 2025 update, 15 seconds for all users, up to 25 seconds on the ChatGPT Pro web version; the Sora 2 Pro API supports 10s/15s/25s tiers. There is no information on how training-side clip durations are bucketed or how clips are segmented from longer videos for training. (The prior generation Sora 1 stated that it trained at native duration without uniformly cropping to a fixed frame count; whether Sora 2 continues this is unconfirmed.) [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

Training-side distribution and bucketing strategy are undisclosed. Inference-side specs: the standard Sora 2 is 720p, while Sora 2 Pro supports 1024p and native 1080p (1920x1080), with both portrait (1080x1920) and landscape output supported. The prior generation Sora 1's technical blog explicitly adopted "native size training" — no resizing/cropping/trimming to a fixed size, allowing native sampling of arbitrary aspect ratios; it is presumed that Sora 2 continues this strategy (variable-resolution/aspect-ratio patch packing), but this has never been confirmed by Sora 2's official materials, and there are no bucket-proportion figures whatsoever. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

Completely undisclosed. The System Card gives no proportions for any category — people/actions/scenes/styles — does not describe a concept-balancing strategy, and does not disclose how long-tail concepts are handled. The only indirect clue comes from capability descriptions: the model is emphasized as performing better on physical laws (gravity, momentum, buoyancy, material deformation, collision dynamics, object permanence); a third-party analysis claims the training data carries "physical annotations" covering these concepts, hinting at a possible targeted data mixture or annotation scheme for physical interaction — but this claim comes from a secondhand technical analysis, not an official OpenAI statement, and no proportion figures are given at all. Additionally, from the product form (cameo appearances, social feed), it can be inferred that the proportion of person/face data is likely not low, but again this has no official basis. [Uncertain]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effects foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Completely undisclosed. As a native audio-video model, Sora 2's capabilities explicitly cover four audio categories — dialogue, sound effects/foley (sound effects tied to on-screen actions), background music (matched to scene tone), and ambient sounds/atmosphere (context-aware background soundscapes) — and it claims that sound volume and spatial positioning vary with the distance of objects and the camera. But OpenAI gives no information at all on what proportion of the training data each of these four categories occupies, how tracks with no audio/silent clips are handled and retained, or whether there is explicit mixture control across speech-effects-music. This is one of the largest information gaps in this research: the model clearly has this capability, but the data-side construction method is entirely undisclosed. [Uncertain]

### Narrative structure distribution (single-shot vs multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

The shot-count distribution and single-/multi-shot mixture of training data are undisclosed. Capability-side disclosure: Sora 2 can "follow intricate instructions spanning multiple shots while accurately persisting world state," i.e., it supports instruction-following across multiple shots while maintaining consistency of characters, environment, and lighting across shot changes, indicating that the training data must necessarily include multi-shot narrative samples with cross-shot consistency supervision signals. The Storyboard tool launched in October 2025 further allows users to plan multi-scene videos segment by segment. As for whether native audio tracks are included: from the native joint audio-video generation capability, it can be inferred that the training data is primarily composed of "video with synchronized original audio," but the average clip duration, shot-count histogram, and the ratio of samples with/without audio are all without figures. [Uncertain]

### Language/accent distribution (data basis for multilingual lip-sync capability) ⚠️

Completely undisclosed. The model has dialogue-generation and lip-sync capabilities, and in practice can generate speech in multiple languages, but OpenAI has not published a list of supported languages, per-language data proportions, or accent distribution, nor has it explained the data basis for multilingual lip sync. On the safety side, only "audio transcripts" are mentioned as being passed through safety classifiers, indirectly indicating the existence of an ASR capability, but this does not address language coverage. [Uncertain]

## Cleaning Pipeline

### Overall funnel structure (number of filtering stages, order of stages) ⚠️

The training data cleaning funnel structure is completely undisclosed. The System Card's entire statement about cleaning is just two sentences: "Our data processing pipeline includes rigorous filtering to maintain data quality and mitigate potential risks. We also employ a combination of safety classifiers to help prevent the use or generation of harmful or sensitive content, including explicit materials such as sexual content involving a minor." That is, it only acknowledges the existence of two broad categories — (1) quality filtering and (2) risk/safety classifiers — without describing the number of stages, their order, or the criteria at each stage. By comparison, the System Card's description of the "inference-time safety stack" (input prompt blocking → generation → output blocking, including a CSAM classifier and a custom-trained multi-modal reasoning monitoring model) is far more detailed — this is a defining characteristic of this model's disclosure pattern: detailed on safety and deployment, nearly blank on training data. [Uncertain]

### Quantitative funnel retention rates (input/output volume and final retention rate at each filtering stage, e.g., Apollo 27%) ⚠️

Completely undisclosed. There are no input/output volumes, retention rates, or elimination rates for any filtering stage. It cannot be compared to a publicly disclosed quantitative funnel such as Apollo's 27%. [Uncertain]

### Shot segmentation method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

Completely undisclosed. Neither open-source tools such as PySceneDetect nor an in-house shot-aware splitting model are mentioned. Although the model has multi-shot generation capability and cross-shot world-state persistence — implying that shot-level segmentation and shot-organization logic exist somewhere in the training pipeline — the method is entirely undisclosed. [Uncertain]

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

Only the single vague statement "rigorous filtering to maintain data quality" exists. Specific means such as an aesthetic scoring model, sharpness/blur criteria, OCR text/subtitle filtering, black-bar cropping, and watermark/logo detection are not mentioned by OpenAI at all, with no threshold or model name given. [Uncertain]

### Motion filtering (optical-flow/motion-score thresholds, removal of static and shaky footage) ⚠️

Completely undisclosed. No mention of optical-flow computation, motion-score thresholds, static-shot removal, or shaky/handheld-footage removal strategy. Given that the model's performance in physical dynamics (gravity, momentum, collisions) is heavily promoted, some form of motion-quality screening is presumed to exist, but there is no basis for this. [Uncertain]

### Deduplication methods (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Completely undisclosed. Neither exact deduplication (hash-level) nor embedding-based semantic deduplication is described. [Uncertain]

### VLM/LLM as data quality judge (multi-modal large model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

Whether a VLM/LLM is used as a data quality judge on the training-data side is undisclosed. However, on the inference safety side, OpenAI clearly uses a large model as a judge: in the output-blocking stage it deploys "a safety-focused reasoning monitor... a multimodal reasoning model which is custom-trained to reason about content policies," making semantic-level judgments on generated video frames, scene description text, and audio transcripts. This proves OpenAI has already built and deployed a "large-model semantic judgment" capability stack, and it would be technically natural to reuse similar capability for training-data quality inspection, but the System Card makes no such statement. Additionally, in the safety evaluation process, adversarial prompts are used to generate outputs from a "helpful-only version of the video model," which are then scored and converted into automated evaluation, also reflecting a paradigm where the model participates in data construction and judgment. [Uncertain]

### Safety and compliance filtering (NSFW, copyright, face/privacy)

This is the only training-data-side cleaning step with substantive disclosure. Explicit content: (1) uses "a combination of safety classifiers" to prevent harmful or sensitive content from being used or generated, explicitly naming content involving minors (CSAM); (2) on child safety, it takes the approach of "responsibly sourcing datasets to exclude CSAM" — i.e., filtering out CSAM at the data source, in partnership with NCMEC, and applying strong scanning to all inputs and outputs (including first-party products and API/enterprise third-party use), except where customers meet strict exemption criteria; (3) it has a dedicated CSAM safety stack, reusing system-level mitigations from other products while layering on Sora-specific protections. NSFW/copyright/face-privacy are not separately described as training-data-side filtering methods, and are mainly handled via deployment-side policy: no video-to-video support, no text-to-video of public figures, blocking generation involving real people except for users who have explicitly opted in via cameo, and applying stricter thresholds to uploaded material suspected of depicting minors.

## Annotation Methods

### Caption model used (in-house VLM/open-source model, model scale) ⚠️

Completely undisclosed. Sora 2 has not published any caption-model information (in-house VLM name, scale, or whether GPT-4o/GPT-5-series models are used for annotation). For reference, the prior generation: Sora 1's technical blog stated that a "highly descriptive captioner model" was trained to produce highly descriptive text captions for training videos (following the re-captioning approach used with DALL·E 3), and at inference time GPT was used to expand users' short prompts into long, detailed captions. Sora 2 very likely continues and upgrades this approach, but there is no official confirmation and no model-scale figures. [Uncertain]

### Caption density and structuredness (short/long/dense descriptions, structured fields such as camera motion, style tags) ⚠️

Completely undisclosed. There is no information on caption length distribution, density, or whether structured fields (camera motion, composition, style tags, etc.) are used. The prior generation Sora 1 emphasized that "highly descriptive captions" improve text fidelity and video quality, from which it can be inferred that Sora 2 likely still follows a long, dense-caption approach; and from capability claims such as "enhanced steerability," "expanded stylistic range," and "follows intricate instructions spanning multiple shots," it can be inferred that the caption likely includes style tags, camera/movement descriptions, and multi-shot segmentation structure, but all of this is speculative. [Uncertain]

### Joint audio-video caption structure (whether visual and auditory tracks are both covered, whether split into independent fields — e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field scheme) ⚠️

Completely undisclosed. As a native audio-video model, Sora 2 must necessarily need annotation covering the auditory track (otherwise it could not respond to text instructions specifying dialogue/effects/music); in actual use, users can directly specify line content via quoted dialogue in the prompt, indicating that a clear dialogue field or an equivalent mechanism exists in the caption schema. But OpenAI does not state whether it uses a single fused caption, or splits into independent visual/dialogue/sound-effect fields as with LTX-2 (full soundscape description), Script-a-Video (factorized streams), or Foley-Omni (three-field scheme). This is the largest information gap when comparing against equivalent open-source work. [Uncertain]

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Undisclosed on the training-data side. Indirect evidence: the safety stack explicitly feeds "audio transcripts" as a separate channel into safety classifiers, indicating that OpenAI possesses and routinely uses ASR transcription of video audio tracks — a capability that would be a natural extension to apply to training-data annotation as well. Attributes such as speaker identity/language/accent/emotion annotation are not disclosed at all. On the product side, the cameo feature requires users to record a one-time video+audio sample to complete identity verification and bind their likeness and voice, indicating the existence of a "speaker identity-timbre-likeness" binding representation, but this belongs to a personalization/conditioning-injection mechanism rather than a training-data annotation schema. [Uncertain]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action labels, explicit state) ⚠️

Completely undisclosed. No mention of camera parameters, depth maps, 3D point tracks, action labels, or explicit physical-state annotation. A third-party technical analysis claims the training data carries "physical annotations" covering gravity, momentum, buoyancy, material deformation, and collision dynamics; if true this would constitute explicit structured state annotation, but this claim is not an official OpenAI statement and cannot be verified. OpenAI only officially claims, at the capability level, that the model "understands real-world physics" and positions Sora 2 as a step toward a physical world simulator. [Uncertain]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV) ⚠️

Completely undisclosed. The word "generate" in the System Card's phrase "information that our users or human trainers and researchers provide or generate" may cover content generated by human trainers, but whether it includes model-synthesized data, or whether training pairs are constructed via controlled perturbation/editing (in the manner of InstructAV2AV-style editing pairs), is entirely without information. During safety evaluation, a "helpful-only version of the video model" was used to batch-generate adversarial-sample outputs for building an automated evaluation set — a clear case of the model generating data for evaluation/safety alignment purposes, but this does not belong to the main training data synthesis. [Uncertain]

### Degree of human involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

On the training-data annotation side: only that data "provided or generated by human trainers and researchers" exists is known, indicating human involvement in providing and generating data, but scale, process, and whether manual quality review occurs are all undisclosed. On the deployment side, human involvement is clearly described: collaboration with external members of the OpenAI Red Team Network to test (covering sexual content, nudity, extremism, self-harm, illegal behavior, graphic violence, political persuasion, as well as dedicated policies for minor safety and likeness use); red-team feedback is used to adjust prompt filters, blocklists, and classifier thresholds; content moderation uses a combination of "automation + manual review" to identify patterns of abuse, and in-app reporting and appeal channels are provided. [Uncertain]

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment) ⚠️

The training-data-side audio-video synchronization detection method is completely undisclosed — this is the core concern of this research, and it is precisely where OpenAI discloses nothing. The System Card contains no statement about lip-sync detection, event-alignment detection, or removal of asynchronous samples. On the capability side, it is only claimed that audio is "properly synchronized with on-screen action, including accurate lip-sync for speaking characters," as well as sound effects tied to on-screen events and music tempo matching scene pacing. It can be confirmed that the training pipeline must include some form of AV synchronization quality control (otherwise lip sync could not be learned), but the method, module, and criteria are all unknown. [Uncertain]

### Specific synchronization detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5) ⚠️

Completely undisclosed. No name is given for SyncNet, AV-align, or any in-house synchronization metric, and no confidence-threshold values are given at all (there is nothing to compare against, e.g., UniTalking's SyncNet conf>0.9). OpenAI has also not published any Sora 2 scores on third-party AV synchronization evaluation benchmarks. [Uncertain]

### Separate handling of temporal vs. semantic synchronization (time alignment and content-semantic matching treated as two independent filtering conditions) ⚠️

Completely undisclosed. There is no information on whether "temporal alignment" and "content-semantic matching" are split into two independent filtering conditions. From the capability description, the model appears to possess both temporal alignment (lip movements frame-aligned with speech, sound effects aligned with the moment of collision) and semantic matching (ambient sound fitting the scene context, music mood matching the on-screen tone), hinting that the data side may involve separate handling — but this is pure inference. [Uncertain]

### Audio quality filtering (SNR, silence detection and silence-ratio threshold, no-audio-track removal, off-screen-source removal, background-music separation) ⚠️

Completely undisclosed. No mention of a signal-to-noise-ratio threshold, silence detection and silence-ratio threshold, a strategy for removing samples with no audio track, removal of voiceover/narration sources, background-music separation (source separation), or any other means. [Uncertain]

### Classification and separate handling strategy for speech/sound effects/music ⚠️

Completely undisclosed. On the output side, the model clearly distinguishes dialogue, sound effects/foley, music, and ambient sound as four categories, but whether the training-data side applies explicit classification across these four categories and designs separate filtering/mixture/annotation strategies for each is not explained by OpenAI at all. [Uncertain]

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

Completely undisclosed. There is no information on stage division (low-res→high-res, image→video, short→long, silent→audio, etc.), and no description of scheduling a data curriculum by resolution/duration/quality score/modality. An indirect clue that can be inferred: the product is split into two tiers, Sora 2 (720p) and Sora 2 Pro (1080p), and the duration cap was progressively relaxed from 10 seconds to 15/25 seconds, consistent with the general pattern of a progressive resolution- and duration-based curriculum — but this belongs to product tiering and inference configuration, and cannot be directly equated with a training curriculum. [Uncertain]

### Data mixture changes across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

Completely undisclosed. There is no information on how the data mixture changes across pretraining/annealing/SFT stages. [Uncertain]

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

The post-training data used for training is completely undisclosed. There is no SFT curated-set scale or selection criteria, no number of preference pairs or annotation method, and no description of reward-model training data. The only relevant public information belongs to safety-alignment evaluation rather than model-capability post-training: OpenAI collected "thousands of adversarial prompts" through targeted red-teaming, classified by use case and policy domain, generated outputs using a helpful-only version of the video model, scored them, and converted them into an automated evaluation set to measure two metrics for the production safety stack — not_unsafe (blocking recall) and not_overrefuse (avoiding over-blocking). Published results: adult nudity/sexual content (not involving likeness) 96.04%/96.20%; adult nudity/sexual content (involving likeness) 98.40%/97.60%; self-harm 99.70%/94.60%; graphic violence 95.10%/97.00%; prohibited political persuasion 95.52%/98.67%; extremism/hate 96.82%/99.11%. [Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Completely undisclosed. No mention of NeMo Curator, Data-Juicer, or an in-house data-processing framework, and no figures for GPU acceleration ratio, processing scale, or cost. The only indirect economic clue comes from media reports: Sora's service is reported to cost roughly $15 million per day to run while generating only about $2.1 million in revenue, and active users fell below 500,000 in early 2026 — considered the main reason OpenAI shut down the Sora consumer app in March 2026. This reflects inference-side compute economics rather than data-processing throughput. [Uncertain]

## Performance Comparison

### Quantitative impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

Completely undisclosed. The System Card contains no ablation experiments at all — no filtering-strictness ablation, no caption-density/style ablation, no data-mixture ablation, and no corresponding table of evaluation-metric comparisons. OpenAI has not published any official Sora 2 scores on VBench or any other public video-generation benchmark. The only quantitative table in the System Card is the six-category safety-metric table mentioned above. [Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data outperforms large, messy data) ⚠️

Completely undisclosed. There is no experimental evidence or discussion of "small, refined data outperforming large, messy data." The System Card offers only the single qualitative statement "rigorous filtering to maintain data quality," which implies a stance of valuing data quality, but there is no supporting evidence of any kind. [Uncertain]

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g., VABench's seven major categories) ⚠️

Completely undisclosed. OpenAI has not published a domain taxonomy for Sora 2's training data, nor has it aligned that taxonomy with any evaluation benchmark's categories (e.g., VABench's seven categories, VBench, etc.). The only taxonomy OpenAI has built is a safety-policy taxonomy (adult sexual content/self-harm/graphic violence/political persuasion/extremism-hate, plus a binary split on whether likeness is involved), which serves safety evaluation and has no correspondence to training-data domain distribution. [Uncertain]

## Other Information

### summary_note

Core conclusion: Sora 2 is the representative sample with the lowest degree of data disclosure under this topic. Its System Card is 7 pages total, of which content touching on training data is confined to a single paragraph in Section 2 (about 5 sentences, not a single number), while the safety stack, usage policy, provenance tools, red-teaming, and safety evaluation occupy roughly 85% of the document. Across all 42 researched fields, only 8 — name/organization/release_date/type/openness/av_generation/provenance_licensing/safety_filtering — can be substantively answered based on official material; the remaining 34 (covering data scale, the cleaning funnel, the caption schema, audio-video alignment detection, the training curriculum, ablation evidence, and every other technical dimension) have no official information at all. OpenAI has also never published a Sora 2 technical report or paper, continuing the practice — dating back to GPT-4 — of "not disclosing model and implementation details." As a result, this entry has very limited reference value at the level of data-processing methodology; its main research value lies in: (1) serving as a capability benchmark and product-form example of native joint audio-video generation; (2) serving as a typical case of the asymmetric disclosure pattern of "detailed output-side provenance governance (C2PA + watermark + internal detection) vs. blank training-side data provenance"; (3) its abrupt copyright opt-out→opt-in policy reversal, and the controversy over opt-in not applying retroactively to training data, is an important case study in training-data compliance issues. For reproducible audio-video data-processing methodology, one should instead turn to LTX-2, Ovi, MMAudio, Veo 3 technical materials, and related open-source work.

## Uncertain Fields

The research information for the following fields is partially uncertain (sources marked ⚠️):

- av_generation
- data_scale
- data_sources
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
