# InstructAV2AV (paired dataset InsAVE-80K). The paper's full title is "InstructAV2AV: Instruction-Guided Audio-Video Joint Editing," i.e., instruction-guided joint audio-video editing. Its core positioning is as the first end-to-end framework that "edits both the video frames and their accompanying audio track using only text instructions" — changing the specified visual target and its produced sound while strictly preserving the background footage and ambient sound in non-target regions. Its representative dimension within this survey's lineage is "synthetic data": the paper builds a scalable data synthesis pipeline (data engine) that artificially manufactures source→target paired samples through controlled editing, solving the fundamental predicament in the audio-video editing field that "paired supervision data simply does not exist."

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipelines, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Method](#annotation-method) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Results Comparison](#results-comparison)

## Basic Information

### Name

InstructAV2AV (paired dataset InsAVE-80K). The paper's full title is "InstructAV2AV: Instruction-Guided Audio-Video Joint Editing," i.e., instruction-guided joint audio-video editing. Its core positioning is as the first end-to-end framework that "edits both the video frames and their accompanying audio track using only text instructions" — changing the specified visual target and its produced sound while strictly preserving the background footage and ambient sound in non-target regions. Its representative dimension within this survey's lineage is "synthetic data": the paper builds a scalable data synthesis pipeline (data engine) that artificially manufactures source→target paired samples through controlled editing, solving the fundamental predicament in the audio-video editing field that "paired supervision data simply does not exist."

### Publishing Organization/Company

Beijing Academy of Artificial Intelligence (BAAI) in collaboration with Peking University (PKU). Author list: Haojie Zheng (郑浩杰, first author, dual affiliation with BAAI and PKU, personal homepage hjzheng.net), Yixin Yang (PKU), Siqi Yang (PKU), Shuchen Weng (翁书宸, dual affiliation with BAAI and PKU), Boxin Shi (施柏鑫, PKU School of Computer Science, corresponding author, working in vision and computational photography). The code repository is hosted under the personal GitHub account suimuc/InstructAV2AV, with HuggingFace account suimu.

### Release Date (Technical Report/Paper/Open-Source Release) ⚠️

Submitted to arXiv on May 18, 2026 (arXiv:2605.18467, v1). The project homepage https://hjzheng.net/projects/InstructAV2AV/, GitHub repository suimuc/InstructAV2AV, HuggingFace model suimu/InstructAV2AV, and dataset suimu/InsAVE-80K all went live at the same time. As of the time of this survey (July 2026), inference code, weights, and dataset have all been released; the training script is still marked as in progress on the roadmap. [Uncertain] No formal conference/journal acceptance information found; currently in arXiv preprint status.

### Type (Model/Dataset/Toolchain/Evaluation Benchmark)

A trinity output of model + dataset + data synthesis pipeline (data engine), rather than a pure video generation model.
[Model] InstructAV2AV, an instruction-guided audio-video joint editing model adapted from the Ovi symmetric twin-tower DiT architecture, taking "source audio-video + text instruction" as input and outputting "edited audio-video." Note that its task is editing rather than generation from scratch.
[Dataset] InsAVE-80K, which the paper claims is the first large-scale audio-video editing dataset, containing source-to-target paired samples.
[Pipeline] The scalable data synthesis engine described in Section 3 of the paper is this work's primary contribution on the data dimension, and is the focus of this survey.
[Evaluation] No independently named evaluation benchmark is proposed; instead, 1K manually curated samples are carved out of InsAVE-80K as an evaluation set, with additional zero-shot generalization evaluation performed on the existing AvED-Bench.

### Openness (Whether Weights/Code/Data/Pipeline Are Each Open-Sourced) ⚠️

Relatively high degree of openness — one of the most data-open audio-video works covered in this survey.
[Weights] Open-sourced. HuggingFace suimu/InstructAV2AV releases pretrained weights and provides six checkpoints fine-tuned for different subtasks (covering general editing, content addition/removal, identity/voice cloning scenarios, etc.). [Uncertain] The model repository page has no complete model card; parameter count, license, etc. are not labeled on the model page.
[Code] Partially open-sourced. The GitHub repository releases inference code (scripts/edit.py, scripts/demo.py) and pipeline scripts under an Apache-2.0 license; training scripts are listed as "in progress" on the README roadmap and have not yet been released. Depends on upstream Wan-AI/Wan2.2-TI2V-5B and hkchengrex/MMAudio.
[Data] Fully open-sourced — this is the most standout openness point of this work. HuggingFace suimu/InsAVE-80K is released under an MIT license, actually packaging 88,074 video pairs (176,148 files), split into 11 tar shards, totaling about 139 GB, distributing the actual video and audio files rather than just URLs — this is relatively rare among audio-video datasets (compare Foley-Omni's V2ST-Bench, which releases only URLs + metadata due to copyright restrictions). The data card also warns users that they must independently verify the rights compliance of the underlying media content.
[Pipeline] Methodologically well-disclosed: it fully lists the specific models used at every stage (PySceneDetect, CoTracker3, LAION Aesthetics Predictor, PyDub, Audiobox-Aesthetics, TalkNet, ElevenLabs Scribe, Grounded-SAM-2, Qwen3-Omni, SAM-Audio, ElevenLabs TTS, Wan2.2-5B), but the specific numerical values of most filtering thresholds are not given (only -45 dBFS has an explicit number), and there is no stage-by-stage retention rate, so reproducibility is limited.

### Whether Joint Audio-Video Generation Is Supported, and Implementation Approach (Native Joint/Cascaded/MoE Fusion)

Supports joint audio-video generation, but the task form is "joint editing" rather than "joint generation from scratch," and the implementation approach is native joint rather than cascaded.
[Model-side architecture] Based on Ovi's (Low et al., 2025) symmetric twin-backbone diffusion transformer: the video tower and audio tower run in parallel, achieving joint denoising via cross-tower interaction; the video side encodes/decodes with a spatial-temporal VAE, the audio side with a 1D VAE, and the text side encodes instructions with T5. Audio and video are produced synchronously within the same diffusion process, naturally guaranteeing temporal alignment, rather than the cascaded approach of "generate video first, then dub."
[Three adaptations for the editing task]
  1. Source Concatenation (SC): concatenates the latent of the source audio-video with the noise latent along the channel dimension, anchoring the source context for the generation process — the core mechanism for keeping non-target regions unchanged. Ablation shows that removing it worsens FVD from 180.38 to 467.20 (a 1.6x degradation), with severe background degradation — the most impactful of the three designs.
  2. Source-Instruction Gated Attention (SIGA): soft gated attention between source information and instruction information, balancing the conflicting goals of "follow the instruction to change" and "preserve the original content." Ablation shows removing it raises FAD from 2.75 to 3.26, producing audio hallucinations and stuttering.
  3. Two-Stage Training Strategy (TSTS): first adapts each modality separately, then jointly fine-tunes, used to smoothly transfer pretrained priors. Ablation shows removing it raises FVD to 291.55 and FAD to 5.18, producing visual distortion and inconsistency.
[Loss weighting] Joint training uses modality-balancing weights λ_v = 0.85, λ_a = 0.15, with the visual-side weight far higher than the audio side (about 5.7:1), reflecting that the visual branch is harder to converge.
[Audio-video coupling on the data-engine side] Worth noting separately: the mask-guided video editing model (based on Wan2.2-5B) used to construct training data itself also has an audio-video coupling design — injecting audio features into the video synthesis process via frame-wise cross-attention, to ensure the synthesized target video is strictly temporally synchronized with the already-synthesized target audio. That is, "audio-video alignment is already enforced at the synthetic-data stage," rather than filtered after the fact.

### List of Research Information Sources (URLs of papers/technical reports/official docs/news, with each source's nature noted: primary official/same-team corroboration/third-party report)

1. [Primary official] arXiv abstract page https://arxiv.org/abs/2605.18467 — title, authors, abstract, submission date of May 18, 2026.
2. [Primary official] arXiv HTML full text https://arxiv.org/html/2605.18467v1 — Section 3 data synthesis pipeline, method design, experiment tables, ablations, user study, limitations. The core source of information.
3. [Primary official] arXiv PDF https://arxiv.org/pdf/2605.18467
4. [Primary official] Project homepage https://hjzheng.net/projects/InstructAV2AV/ — demos for four editing task types (identity-preserving speech modification / AV instance editing / instance insertion / instance removal), comparison examples with AvED, CoherentAVEdit, AVI-Edit, and resource links.
5. [Primary official] GitHub repository https://github.com/suimuc/InstructAV2AV — Apache-2.0 license, inference and pipeline scripts, six subtask checkpoints, training script roadmap status, upstream dependency declarations.
6. [Primary official] HuggingFace dataset https://huggingface.co/datasets/suimu/InsAVE-80K — 88,074 pairs, 139 GB, 11 shards, five training subset names (add_and_remove / clone_id / clone_id_voice / clone_voice / general_editing), CSV field structure (original_video / target_video / instruction / instruction_reverse), <S>/<E> speech markers, MIT license. This is the key corroborating source that fills in data organization details not disclosed in the paper.
7. [Primary official] HuggingFace model https://huggingface.co/suimu/InstructAV2AV — weight release, but no model card.
8. [Third-party report] X/Twitter @wildmindai post https://x.com/wildmindai/status/2058634841013285372 — a popular summary of the "Ovi + Wan2.2 + T5" tech stack and mask-free object/sound replacement capability, no new primary data.
9. [Third-party index] The 500 Feed https://www.the500feed.com/story/e7f8eb37a6ffa09c, ai-search.io tool page https://ai-search.io/tool/instructav2av — aggregated introductions, no new information.
10. [Third-party] YouTube demo video https://www.youtube.com/watch?v=0qp0B4jkWjE

## Data Scale and Distribution

### Training Data Scale (Video Count/Hours/Token Count, Pretraining vs. SFT Separated) ⚠️

[Final dataset scale] Per the paper: InsAVE-80K contains 79K automatically verified training pairs + 1K manually curated evaluation pairs, totaling about 80K (hence the dataset name).
[Discrepancy from the actually released figures] The HuggingFace dataset card shows an actual release of 88,074 pairs (176,148 files, since each pair contains both a source and a target), of which 87,074 are training pairs and 1,000 are evaluation pairs, totaling about 139 GB across 11 tar shards. That is, the actual released volume exceeds the paper-reported 79K by about 8K pairs (roughly 10% more), presumably because the pipeline continued running and expanding after paper submission, or possibly a rounded figure in the paper. The actual HF release numbers should be used as the reference.
[Per-sample specification] Uniformly 5-second clips, 720p resolution, 24 FPS, 16 kHz audio. Converting 79K×5s, the total training duration is about 110 hours (source side); counting the target side as well, about 220 hours. This scale is extremely small relative to the tens of thousands to hundreds of thousands of hours typical for text-to-video pretraining, but is reasonable for the post-training nature of an "editing" task — the model's generation capability comes from Ovi's pretraining prior, and InsAVE-80K is only responsible for teaching the model "how to edit according to instructions."
[Pretraining vs. SFT split] The paper does not split InsAVE-80K into pretraining/SFT portions. The true pretraining data is the upstream Ovi and Wan2.2 training corpora (not covered or disclosed in this paper); InsAVE-80K as a whole plays the role of an editing-task fine-tuning set.
[Evaluation scale] The InsAVE-80K evaluation set has 1,000 pairs (manually curated); additional zero-shot evaluation is performed on the external AvED-Bench. The user study randomly draws 20 samples from each dataset.
[Uncertain] The paper does not disclose the size of the raw video pool before data synthesis (hours or clip count), nor how many candidate samples the data engine actually generated before filtering down to 79K, so the absolute throughput and filtering intensity of the whole pipeline cannot be quantified.

### Data Source Composition (In-House/Public Datasets/Web Scraping/Licensed Procurement/Synthetic Data)

Structurally a two-stage composition of "public datasets + web scraping → controlled synthesis" — this is the most fundamental difference between this work and other subjects of this survey: the target side of the final training data is entirely model-synthesized product, not genuinely filmed footage.
[Stage one: real-material sources (used as source and synthesis base)]
  · Web platform scraping: explicitly mentions YouTube (the paper phrases it as "publicly accessible online platforms, e.g., YouTube").
  · Four public datasets: MovieBench (Wu et al., 2025b, a film-level long-video understanding dataset), Condensed Movies (Bain et al., 2020, a collection of key movie clips), Short-Films-20K (Ghermi et al., 2024, a short-film dataset), VGGSound (Chen et al., 2020, about 310K in-the-wild 10-second audio-video event clips, 310 sound-source classes).
  The orientation of the source composition is clear: the three film/short-film sources provide narrative, dialogue-bearing, professionally mixed high-quality material (supporting speech editing and identity-preservation tasks), while VGGSound provides material with clear sound-source events (supporting sound-effect-type instance editing/insertion/removal tasks). This "film + event" dual configuration corresponds one-to-one with the four editing task types.
[Stage two: synthetic data (target side)]
  Target videos are synthesized by a self-built mask-guided editing model (based on Wan2.2-5B); target audio is produced by SAM-Audio separation + ElevenLabs text-to-audio/speech synthesis + mixing with the original background sound. Thus in each pair, the source is real and the target is synthetic, and the instruction is generated by Qwen3-Omni.
[Licensed procurement] None. No paid licensing or rights-cleared data procurement is mentioned.
[In-house data] None. No internal private data is mentioned.
[Methodological significance] This approach of "anchoring on real material, using a generative model to manufacture controlled changes" is a general paradigm for solving the problem of nonexistent paired supervision data: in the real world, there is no such thing as "two genuine recordings of the same scene, before and after editing" — it can only be synthesized. The cost is that the target side inherits the capability ceiling and distortion patterns of Wan2.2-5B and ElevenLabs; the editing effects the model learns cannot exceed the data engine itself — this is an inherent ceiling of the synthetic-data route, which the paper does not discuss.

### Data Compliance and Provenance (Proportion of Licensed Data, Rights-Cleared Datasets, C2PA, etc.) ⚠️

[Uncertain] Compliance and provenance disclosure are weak, and there is an evident open-vs-compliance tension.
[Known positive practices] The dataset is released under an MIT license, and the data card explicitly warns users that they must verify compliance with rights applicable to underlying media content — that is, the authors shift copyright responsibility to downstream users. The code side is Apache-2.0, and explicitly declares dependence on upstream components such as Wan-AI/Wan2.2-TI2V-5B and hkchengrex/MMAudio.
[Existing compliance risks]
  · Direct distribution of media itself: unlike the academic convention adopted by VGGSound, AudioSet, Foley-Omni's V2ST-Bench, etc., of "releasing only URLs + metadata to avoid redistribution risk," InsAVE-80K directly packages and distributes 139 GB of video/audio files. Its source material includes YouTube-scraped content and film/short-film datasets such as Condensed Movies, MovieBench, and Short-Films-20K, and this underlying material is itself largely copyright-protected; the legal risk of direct redistribution is significantly higher than URL distribution. Although the target side is synthetic, the source side remains original material.
  · Identity and voiceprint issues: the dataset contains three subsets — clone_id, clone_voice, clone_id_voice — that explicitly involve identity cloning and voice-timbre cloning of persons. Using ElevenLabs to synthesize speech and TalkNet to locate speakers is, in essence, performing replacement and re-synthesis on the faces and voiceprints of real people. The paper does not mention any portrait-rights/voice-rights licensing, speaker informed consent, or de-identification measures.
[Missing items] No disclosure of the proportion of licensed data, no mention of rights-cleared datasets, no mention of C2PA or any content-credential/watermarking/provenance mechanism, and no detectable marking applied to generated content. Given that the model's direct capability is "swapping people, altering lip movements, changing dialogue" (i.e., the technical definition of deepfakes), the lack of watermarking and provenance mechanisms is the most notable gap in this work's compliance dimension; the paper's limitations discussion also focuses solely on technical shortcomings such as physical realism, lighting consistency, and 3D spatial consistency, without addressing misuse risk.

### Clip Duration Distribution and Segmentation Strategy ⚠️

The distribution is extremely concentrated — all clips are uniformly a fixed 5 seconds, with no distribution to speak of. This is a typical feature of synthetic datasets relative to genuinely captured datasets: since the target is synthesized by a generative model, and the base models Wan2.2-5B / Ovi have a fixed generation window, the output duration is inevitably pinned to the model's native window (5 seconds @ 24 FPS = 120 frames).
[Segmentation strategy] Two levels:
  1. Shot-level segmentation: PySceneDetect (Castellano, 2024) is used to cut the raw long videos scraped/collected into single-shot clips. This step ensures no shot changes occur within a clip, and is the prerequisite for all subsequent processing (point tracking, speaker localization, mask propagation, edit synthesis) to hold — a cross-shot clip would prevent Grounded-SAM-2's instance mask from propagating continuously.
  2. Fixed-length extraction: a 5-second window is taken from each single-shot clip. [Uncertain] The paper does not specify the exact extraction rule — whether it takes the first 5 seconds, the middle 5 seconds, or slides a window to cut multiple 5-second segments (the latter would introduce highly overlapping clips, a potential hidden source of duplication); nor does it explain how single-shot clips shorter than 5 seconds are handled (discarded or padded).
[Comparison with other works in this survey] Foley-Omni uses 5–10 seconds, VGGSound uses a fixed 10 seconds; this work's 5 seconds is on the shorter side among peers, directly limiting the complexity of editable content — 5 seconds can hardly accommodate multi-turn dialogue or complex event sequences, which also explains why all four task types are "localized editing of a single target" rather than narrative-level rewriting.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

Equally highly uniform: all clips are 720p / 24 FPS. Resolution is jointly determined by the data engine's synthesis base (Wan2.2-5B, i.e., Wan2.2-TI2V-5B) and the model's training resolution — a case of "synthesis fixes the specification," so there is no need to bucket the way real data would.
[Quality floor control] Uses the LAION Aesthetics Predictor (Romain and Christoph, 2022) to filter out visually low-quality clips, but [Uncertain] the paper does not give the aesthetic-score threshold value.
[Uncertain] Aspect ratio distribution is not disclosed — judging from the sources, film-type material (MovieBench, Condensed Movies, Short-Films-20K) is mostly widescreen 21:9 or 16:9, while VGGSound and YouTube material is mostly 16:9; it is presumed that the final output is unified to a single 16:9 ratio (720p usually means 1280×720), but the paper does not state whether this involves cropping, scaling, or letterboxing, nor does it mention any bucketing strategy.
[Difference from text-to-video models] Text-to-video models need multi-resolution, multi-aspect-ratio bucketed training to support arbitrary output sizes; this work, being an editing model with fully synthetic data, can use a single specification — an engineering simplification dividend of the synthetic-data route, at the cost that the model's generalization to non-720p/16:9 inputs is unverified.

### Category/Domain Distribution and Mixing Strategy (Ratio Control and Concept Balancing for People, Actions, Scenes, Styles, etc.) ⚠️

The domain-organization dimension of this work is not "visual content category" (people/actions/scenes/styles) but "editing operation type" — this is the fundamental difference between editing datasets and generation datasets, where the object of ratio control shifts from "what was filmed" to "what is being changed."
[The paper's four-task taxonomy]
  (a) Identity-preserving speech modification: changing dialogue content while preserving the speaker's identity/voice-timbre/appearance;
  (b) AV instance editing: changing the appearance features of a person or object in the frame while also changing what it emits sonically;
  (c) AV instance insertion: adding new visual elements and their corresponding sound;
  (d) AV instance removal: deleting visual elements and their corresponding sound.
  The common structure of the four types is "the visual target + its bound sound" changing synchronously, with non-target regions and ambient sound remaining unchanged.
[The five subset names actually released on HuggingFace (finer-grained than the paper)] add_and_remove (addition and removal, corresponding to c+d merged), clone_id (identity cloning, swapping the person only, not the voice), clone_voice (voice-timbre cloning, swapping the voice only, not the person), clone_id_voice (identity + voice-timbre cloned simultaneously), general_editing (general editing, corresponding to b). This five-way split reveals an important design not elaborated in the paper: speech-type editing is further orthogonally decomposed into two independently controllable factors — "visual identity" and "auditory timbre" — and data was constructed for three combinations (changing only one / only the other / both) respectively. This orthogonal-decomposition style of data construction is a unique advantage of synthetic data over real capture — in reality it is almost impossible to capture genuine material of "the same video with the face swapped but the original voice kept"; it can only be manufactured via controlled synthesis, with precise control over which factors change and which don't, thereby giving the model clean, disentangled supervision signals.
[Indirect inference of visual-content domain] Working backward from the sources: MovieBench + Condensed Movies + Short-Films-20K together contribute narrative film scenes (character dialogue, indoor/outdoor scenes, diverse lighting and cinematography), VGGSound contributes in-the-wild sound-source events (animals, instruments, machinery, natural phenomena, etc., 310 classes), and YouTube contributes open-domain material. Overall the material leans toward two categories: "film scenes with people talking" and "event scenes with clearly sounding objects," similar in domain concentration direction to Foley-Omni.
[Implicit domain-narrowing mechanism] CoTracker3 (Karaev et al.) performs grid-based point tracking, discarding clips whose average motion magnitude falls below a threshold — this step excludes purely static, near-motionless scenes, narrowing the domain to clips "with visible motion." The reason is that the editing task requires the target to be trackable by a mask and to exhibit deformation/displacement; static footage lacks editing value and offers no way to verify temporal consistency.
[Uncertain] No quantitative distribution table is given for any category labels: the sample counts of the five subsets, the ratio of visual content categories (people/animals/objects/scenes), the ratio of speech vs. non-speech samples, concept-balancing or long-tail resampling strategies — none of this is disclosed in the paper or the data card. The specific values of the CoTracker3 motion threshold and the LAION aesthetic threshold are also not given.

### Audio Category Distribution and Mixing (Proportions and Control Strategy for Speech/Sound Effects (Foley)/Music/Ambient Sound/Silence) — A Dimension Unique to AV Models ⚠️

Audio categories in this work are organized as a binary split between "speech" and "non-speech sound-source events," with the two branches running through entirely different processing pipelines — this is the clearest branch point in the pipeline's design.
[The binary system and each branch's processing pipeline]
  · Speech-dominated content: TalkNet (Tao et al., 2021) performs active speaker detection, and ElevenLabs Scribe (2025) performs ASR to extract precise speech timestamps. The key constraint is: clips retain speech only when temporally aligned with a visible on-screen speaker — this single rule accomplishes three things at once: excluding voiceover/narration/commentary, excluding post-production dubbing, and ensuring that retained samples naturally come with lip-sync supervision. This treats "sound must originate from a visible on-screen source" as a hard admission criterion for the data, which is more thorough than post-hoc filtering by a sync score.
  · Non-speech content: clips with ambiguous sound sources are filtered out to ensure sound-source clarity, and each retained clip is then assigned a distinct semantic sound-event label (e.g., "dog barking"). "Distinct/unique" is the key constraint — requiring that a clip contain only one dominant sound source; clips with mixed multiple sound sources are excluded. The reason is straightforward: the editing task requires "changing this object's sound without disturbing other sounds"; if a clip's sound sources are mixed, a clean source-target pair cannot be constructed, and SAM-Audio cannot reliably separate them.
[Silence and audio quality] PyDub is used to remove silent clips below -45 dBFS (the only threshold in the paper with an exact value given); Audiobox-Aesthetics (Tjandra et al., 2025) is used for audio quality assessment, discarding low-quality audio (threshold not given).
[The special status of background/ambient sound] In this work, ambient sound is not the object being filtered out, but the object being protected — SAM-Audio (Shi et al., 2025) separates the target entity's sound from the original audio track, and after editing, the new sound is re-mixed with the retained background sound. This means that the background/ambient sound remains strictly consistent between source and target on a per-sample basis, constituting the auditory-side supervision signal for the "content preservation" evaluation dimension. This design differs in purpose from Foley-Omni's use of Bandit for three-way separation: Foley-Omni's separation is to verify annotations, while this work's separation is to construct controlled pairs where "only the foreground sound changes, the background sound does not."
[Music] [Uncertain] The paper does not treat music as a separate processing category, nor does it mention any detection, retention, or removal strategy for background music. Given that the sources include a large amount of film footage (which typically carries a score), the score is likely subsumed under "background sound," separated by SAM-Audio and retained as-is, but this is not explicitly stated.
[Uncertain] The actual quantity or proportion of speech vs. non-speech samples is not given; the category distribution of sound-event labels is not given (whether the 310-class VGGSound label taxonomy is reused is also unstated); no duration-share statistics are given for the various audio categories.

### Narrative Structure Distribution (Single-Shot vs. Multi-Shot, Average Clip Duration, Shot-Count Distribution, Whether Native Audio Tracks Are Included)

All clips are single-shot structures, with no multi-shot samples. PySceneDetect's shot segmentation is the first step of the pipeline, explicitly producing single-shot clips, and the fixed 5-second length further ensures that no editing point falls within a clip.
[Average clip duration] Strictly 5.0 seconds, with no variance.
[Shot-count distribution] Constant at 1, with no distribution.
[Whether native audio tracks are included] The source side always includes a native audio track, which is required — clips with no audio track or that are silent (<-45 dBFS) are removed during filtering. The target-side audio track is a synthetic product (separated background sound + newly synthesized target sound). This constitutes a distinctive property of this dataset: the audio tracks on both sides of each pair share the same background-sound component, differing only in the foreground sound — a precisely controlled differential pair.
[Why single-shot is mandatory] This is not an aesthetic choice but a technical necessity: Grounded-SAM-2's instance masks need to propagate continuously within a clip, CoTracker3's point tracking needs a continuous field of view, and the mask-guided video editing model needs to perform localized replacement within a stable spatiotemporal context — any shot change would simultaneously invalidate all three.
[Limitation] The resulting capability boundary is clear: the model cannot handle editing consistency across shots (e.g., "replace this person throughout the entire film"), nor can it edit narrative segments containing cuts. This is the same issue, viewed from two sides, as the paper's acknowledged limitation of "difficulty maintaining 3D spatial consistency under substantial camera movement."

### Language/Accent Distribution (Data Basis for Multilingual Lip-Sync Capability) ⚠️

[Uncertain] The paper does not disclose language or accent distribution at all — a notable gap in this work's data description, especially given that speech editing is one of its four core tasks.
[Strong circumstantial evidence pointing to English-dominant]
  · The ASR tool is ElevenLabs Scribe, and the speech synthesis tool is ElevenLabs — although both support multiple languages, the paper does not mention any multilingual configuration.
  · The three film-type sources — MovieBench, Condensed Movies, Short-Films-20K — are predominantly English-language film content; VGGSound is YouTube material where the speech component is inherently sparse.
  · Evaluation uses Sync-C / Sync-D (based on SyncNet, vshift=15 frames) to measure lip sync; SyncNet itself is trained on English data and has lower discrimination accuracy for non-English phonemes.
  · The instructions in the dataset's CSV wrap spoken content boundaries with <S> and <E> markers, and the instructions themselves are in English.
[Missing items] No language labels, language-distribution table, accent annotations, or multilingual lip-sync capability evaluation/discussion are seen anywhere. Compared to industrial models like Veo 3, Sora 2, and Kling Omni that emphasize multilingual lip sync, this work makes no claims on this dimension, and its actual capability should be regarded as verified only on English.

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (Number of Filtering Stages, Order)

The entire pipeline can be divided into three major stages and about ten steps; the overall architecture is "first strictly filter real material → then synthesize target pairs under control → finally double verification." Unlike other works in this survey whose "cleaning real data" forms a one-directional funnel, this pipeline's middle section is a generation step, so its shape is an hourglass structure of "funnel + generator + secondary funnel."

[Stage One: Multi-stage audio-visual filtering] The goal is to filter out clean single-shot clips "suitable as an editing base."
  1. PySceneDetect — cuts long videos into single-shot clips, providing the prerequisite for all subsequent mask/tracking operations.
  2. CoTracker3 grid point tracking — computes average motion magnitude, discarding clips below a threshold (removing static footage).
  3. LAION Aesthetics Predictor — removes visually low-quality clips.
  4. PyDub silence detection — removes clips below -45 dBFS.
  5. Audiobox-Aesthetics — removes low-quality audio.
  6. Branch processing: the speech path uses TalkNet to locate the active speaker + Scribe to extract speech timestamps, retaining only clips where "speech is temporally aligned with a visible on-screen speaker"; the non-speech path removes clips with ambiguous sound sources and assigns each remaining clip a unique semantic sound-event label.
  Note the ordering design: visual filtering first, audio filtering second, sound-source attribution judgment last — cost increasing from low to high, a reasonable cost-oriented ordering.

[Stage Two: Data editing engine — the core contribution of this work] The goal is to manufacture (source, instruction, target) triples from a single piece of real material.
  7. Grounded-SAM-2 — obtains instance-level masks, locating editable visual entities.
  8. Qwen3-Omni — conditioned on the instance mask and the source audio-video context, generates diverse text editing instructions covering the four editing operation types.
  9. Audio-side synthesis: SAM-Audio separates the target entity's sound from the original audio track → ElevenLabs text-to-audio/speech model synthesizes a new target sound according to the instruction → seamlessly mixed with the retained background sound to obtain the target audio track.
  10. Visual-side synthesis: a self-built mask-guided editing model (based on Wan2.2-5B, trained with a source-conditioned flow-matching objective) synthesizes new content within the mask region; crucially, frame-wise cross-attention is introduced to inject the already-synthesized target audio features into the video generation process, forcing the visual target to be strictly temporally synchronized with the new audio track (e.g., new dialogue and new lip movement aligned frame by frame).
  The order here is "synthesize audio first, then synthesize video conditioned on the audio," rather than parallel or reversed — because once the duration and rhythm of the speech are fixed, the lip movements must follow it, whereas the reverse is difficult to constrain.

[Stage Three: Dual verification] The goal is to remove samples where synthesis failed.
  11. Automatic verification: a multimodal LLM (Qwen3-Omni) scores along five dimensions — (i) instruction fidelity, (ii) content preservation, (iii) perceptual quality, (iv) audio-video synchronization, (v) safety — retaining only samples that pass all five simultaneously. Produces 79K training pairs.
  12. Manual verification (used only for the evaluation set): 20 volunteers organized into 10 judge pairs, with each candidate sample assessed by 5 independent judge pairs, each pair responsible for one evaluation dimension; a sample must pass all five to enter the evaluation set. Produces 1K manually curated evaluation pairs.

[Methodological assessment]
  Strength one: manufacturing "nonexistent paired supervision data" via controlled generation, and through two mechanisms — SAM-Audio separation-mixing and mask-region restriction — structurally guaranteeing that "non-target regions remain strictly unchanged," so that source and target form a clean differential pair — a precision that real data can never provide.
  Strength two: enforcing audio-video synchronization at the synthesis stage itself via frame-wise cross-attention, i.e., "guaranteeing quality at data-generation time" rather than "filtering after generation," far more efficient than post-hoc filtering by sync scores.
  Strength three: a clear division of labor between automatic and manual verification — automatic verification preserves scale, manual verification preserves evaluation-set trustworthiness, and manual verification adopts the design of "one judge pair per dimension" for independent per-dimension review, avoiding the mutual interference of a single person scoring across multiple dimensions.
  Shortcomings: nowhere in the whole process is a quantitative retention rate disclosed; almost all threshold values are missing except -45 dBFS; there is no deduplication step; there is no data-side ablation to prove the necessity of each pipeline component.

### Quantitative Funnel Retention Rate (Input/Output Volume and Final Retention Rate at Each Filtering Stage, e.g., Apollo's 27%) ⚠️

[Uncertain] Completely lacking quantitative disclosure — the biggest gap in this work's data methodology.
[The only known endpoint] Final output of 79K training pairs + 1K evaluation pairs (87,074 + 1,000 actually released on HF).
[All missing items]
  · The raw video pool size is unknown — it is not stated how many hours or clips were scraped from YouTube, or how many were taken from each of MovieBench/Condensed Movies/Short-Films-20K/VGGSound.
  · The elimination rate of each of Stage One's six filtering steps is unknown — how many clips PySceneDetect cut out, how many CoTracker3's motion filter eliminated, how many LAION aesthetics eliminated, how many the -45 dBFS silence threshold eliminated, how many Audiobox eliminated, how many TalkNet/sound-source attribution judgment eliminated — none of this has data.
  · The Stage Two generation volume is unknown — how many candidate targets the data engine actually synthesized.
  · The Stage Three verification elimination rate is unknown — what proportion of synthesized samples the five-dimension MLLM verification rejected. This figure is especially regrettable to lack: it would directly quantify the data engine's success rate, a key metric for assessing the engineering feasibility of this pipeline (how many attempts are needed to synthesize one usable sample, and at what cost), and is exactly the evidence most needed to support the self-positioning of "scalable data synthesis pipeline." How many candidates the manual verification filtered 1K out of is likewise unreported.
[End-to-end retention rate cannot be computed] Since data is missing at both ends, an overall retention rate analogous to Apollo's 27% cannot be given.
[A rough extrapolation that can be attempted] Drawing on the experience of similar synthetic data engines, the one-pass success rate of diffusion models doing mask-guided local editing is typically in the 30–60% range (failure modes include boundary artifacts, target deformation, temporal flickering); layering on the strict five-dimension verification (must pass all five simultaneously), the Stage Three retention rate is estimated at roughly 20–50%; i.e., producing 79K training pairs may have required synthesizing 150K–400K candidates. This is purely extrapolation, with no data support from the paper.

### Shot Segmentation Method (PySceneDetect/In-House Model/Shot-Aware Splitting) ⚠️

Uses PySceneDetect (Castellano, 2024) as the shot-segmentation tool, placed as the first step of the entire pipeline, cutting raw long videos into single-shot clips.
[Why it is placed first] Shot segmentation in this pipeline is not an optional cleaning step but a hard prerequisite: the subsequent CoTracker3 point tracking, Grounded-SAM-2 mask propagation, TalkNet speaker localization, and mask-guided video synthesis all depend on continuous field of view within a clip; any shot change would break all four simultaneously. By comparison, shot segmentation in text-to-video models is mainly to avoid the model learning abrupt transitions — a much lower priority.
[Uncertain] Specific configuration is not disclosed: the type of detector used (ContentDetector / AdaptiveDetector / ThresholdDetector), threshold parameters, and minimum shot length settings are not stated. Nor is it mentioned whether a learning-based shot boundary detection model like TransNetV2 was layered on for secondary verification — PySceneDetect is based on pixel statistics and has a relatively high false-detection rate on gradual transitions (fades/dissolves) and fast camera movement, and the film material in the sources is precisely rich in such transitions — a potential quality hazard the paper does not discuss.
[Fixed-length processing after segmentation] Single-shot clips are further reduced to a 5-second window; the specific windowing rule is not stated (see duration_distribution).

### Quality Filtering (Aesthetic Scoring, Sharpness, OCR Text Filtering, Letterbox/Watermark/Logo Detection) ⚠️

Quality filtering is distributed across two points at the head and tail of the pipeline, taking the form of "material admission filtering" and "synthetic-product acceptance filtering," with the latter being a step unique to editing-type datasets.

[A. Material admission filtering (Stage One, applied to real material)]
  · Visual aesthetics: LAION Aesthetics Predictor (Romain and Christoph, 2022) removes visually low-quality clips. This is a standard tool in text-to-video cleaning, reused here. [Uncertain] The threshold value is not given (this model typically outputs a score of 1–10, with common industry thresholds in the 4.5–5.5 range).
  · Motion intensity: CoTracker3 grid point tracking computes average motion magnitude, discarding clips below a preset threshold. [Uncertain] The threshold is not given.
  · Audio quality: Audiobox-Aesthetics (Tjandra et al., 2025, Meta's reference-free audio aesthetics evaluation model) removes low-quality audio. [Uncertain] The threshold is not given. Note: Foley-Omni, using the same model, gives an explicit threshold of ≥0.6; this work does not.
  · Sound-source clarity: non-speech clips with ambiguous sound sources are removed, requiring each clip to have a single dominant sound source. This is a relatively distinctive filtering condition, serving the downstream need for "separable, directionally editable" content.
  · Sound-image attribution: speech clips are required to have speech temporally aligned with a visible on-screen speaker, effectively removing voiceover, narration, and post-production dubbing.

[B. Synthetic-product acceptance filtering (Stage Three, applied to the generated target)]
  A multimodal LLM (Qwen3-Omni) scores along five dimensions, requiring all five to pass simultaneously:
  (i) instruction fidelity — whether the editing required by the instruction was strictly executed;
  (ii) content preservation — whether non-target regions/ambient sound remain completely untouched;
  (iii) perceptual quality — whether the synthesized result is realistic and free of artifacts;
  (iv) audio-video synchronization — whether the new audio track and new footage are aligned;
  (v) safety — see safety_filtering.
  These five dimensions are not generic quality scores but correspond one-to-one with the four failure modes of the editing task (what should have changed didn't / what shouldn't have changed did / the change looks unreal / the change is out of sync), a design that fits the task itself quite closely, and echoes the 11 evaluation metrics at the evaluation stage (TV-A/TA-A correspond to i, SSIM/TC to ii, FVD/FAD/LPAPS to iii, AV-A/PEAVS/Sync-C/Sync-D to iv) — a consistent design of "filtering data the same way the model is evaluated."

[Common items not covered] [Uncertain] No mention of OCR text/subtitle filtering, letterbox/black-bar detection, watermark/logo detection, sharpness (blur) detection, or bitrate thresholds. The absence of subtitle filtering is worth noting: the sources include a large amount of film footage, and burned-in subtitles would cause obvious artifacts in the editing task (when the editing region overlaps with subtitles, mask-guided synthesis would corrupt the text) — a risk the paper does not discuss.

### Motion Filtering (Optical Flow/Motion-Score Thresholds, Removal of Static and Jittery Content) ⚠️

Motion filtering is implemented via CoTracker3 (Karaev et al., Meta's point-tracking model), which samples points on a grid layout and tracks them, computing the clip's average motion magnitude; clips below a preset threshold are discarded.
[Distinctive tool choice] Using a point-tracking model rather than traditional optical flow (RAFT/UniMatch) or frame-differencing to measure motion is relatively unusual among the subjects of this survey. The advantage of point tracking is that it gives a long-horizon trajectory-consistency measure rather than instantaneous per-frame motion, better reflecting "whether there is sustained, trackable motion within the clip," and it is highly consistent with downstream needs — the editing task requires the target to be stably trackable and replaceable throughout the entire 5 seconds, and the success of point tracking is itself a proxy for editability. By comparison, optical-flow magnitude cannot distinguish "sustained displacement" from "random jitter."
[One-sided threshold] Only a lower bound is set (removing static content); no upper bound is mentioned. This differs from Foley-Omni's two-sided interval [0.1, 3.2] — this work does not exclude intense motion, because the editing task does not depend on gentle motion magnitude; but judging from the acknowledged limitation of "3D spatial consistency and object permanence being hard to maintain under substantial camera movement," the lack of a motion upper bound may itself be a data-side contributor to this failure mode — if samples with large camera movement were also excluded during data construction, the model might not have exposed this weakness (at the cost of narrower capability coverage).
[Motivation for removing static content] Unlike the motivation of generative models to "avoid generating static footage," static content is removed here because: an editing task on static footage is too trivial (equivalent to image editing), providing no temporal-consistency supervision signal for the model, and offering no way to test audio-video synchronization capability.
[Uncertain] The specific threshold value, grid sampling density, and normalization method for motion magnitude (pixels/frame, or normalized to frame size) are not disclosed, so this setting is difficult to directly reproduce or transfer.

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

[Uncertain] The paper does not mention any deduplication step at all — neither exact deduplication (file hashing, pHash perceptual hashing) nor embedding-based semantic deduplication is described. The dataset card does not mention it either.
[Specific duplication risks under this pipeline, worth noting individually]
  1. Same-source multi-clip duplication: a single film, after PySceneDetect segmentation, can produce dozens to hundreds of single-shot clips; if multiple clips from the same film all pass filtering into the dataset, they will be highly similar in scene, characters, lighting, and mixing style, constituting semantic near-duplicates. The sources MovieBench, Condensed Movies, and Short-Films-20K are all film-level datasets, so this risk is real.
  2. Same-clip multi-edit duplication: the same source clip can naturally have multiple different instructions applied to it, generating multiple targets and thus multiple training pairs. This is a natural and economical practice in data-engine design (reusing the results of expensive upstream filtering), but it causes high duplication on the source side. The paper does not state whether a limit such as "at most N instructions per source" was applied, nor how many distinct source clips the 79K training pairs correspond to — if this number is far smaller than 79K, the actual diversity of the data would be significantly lower than the nominal scale. This is a key unknown for assessing the effective size of this dataset.
  3. Sliding-window segmentation duplication: if the 5-second window is extracted via a sliding window from a long single-shot clip multiple times, adjacent windows would overlap substantially.
  4. Train-evaluation contamination: the 1K manually curated evaluation set and the 79K training set both come from the same pipeline and the same material pool; the paper does not explicitly state that the two have no overlap at the source-material level (it only states that the evaluation set went through triple manual gatekeeping). If different clips from the same film fall into both the training and evaluation sides, evaluation would be biased optimistic. This issue is more serious in this work than in real datasets, because the style of the synthetic target is determined by the same data engine, and what the model learns may be "imitating the output style of Wan2.2-5B + ElevenLabs" rather than general editing ability — and the evaluation set is produced by exactly the same engine — a systematic bias of "evaluation favoring one's own data engine's style" exists that the paper neither discusses nor corroborates with third-party data. Fortunately, the paper additionally performs zero-shot evaluation on the external AvED-Bench (FVD 227.82 / FAD 4.32 / AV-A 23.71, better than AVI-Edit's 372.37 / 7.65 / 23.21), somewhat mitigating this concern.

### VLM/LLM as Quality Inspector (Multimodal LLM Quality Scoring and Mismatch Removal, the 2026 Trend of Shifting from Shallow Scorers to Large-Model Semantic Judgment) ⚠️

The multimodal large model plays a dual role in this pipeline — both "instruction generator" (Stage Two) and "quality judge" (Stage Three) — both roles filled by Qwen3-Omni (Xu et al., 2025b), a typical example of the 2026 trend of "large models deeply involved throughout the entire data-construction chain."

[Role one: instruction generator] Qwen3-Omni, conditioned on the instance-level mask produced by Grounded-SAM-2 and the source audio-video context, generates diverse text editing instructions (formulate diverse text instructions for comprehensive editing operations). Choosing the Omni series (natively supporting text+image+video+audio full-modal input) rather than a pure-vision VLM is necessary — the instruction must be generated based on both "what editable object is in the frame" and "what sound this object makes in the audio track" to be a reasonable joint audio-video editing instruction (e.g., "replace the dog with a cat" implicitly requires "the barking must be replaced with meowing").

[Role two: quality judge] Scores each synthesized target along five dimensions (instruction fidelity / content preservation / perceptual quality / audio-video synchronization / safety), adopting an "AND" logic — retaining only the samples that simultaneously pass all five criteria. This is a strict conjunctive filter rather than a weighted sum, where failing any single dimension results in elimination — a conservative orientation.

[Comparison with Foley-Omni's dual-path verification paradigm] Foley-Omni, after using Gemini 2.5 Pro to generate annotations, deliberately introduces an independent purely acoustic channel (Bandit source separation + energy gating) to veto the large model's hallucinations, on the grounds that "the generation path and the verification path must have decoupled error modes." This work, by contrast, uses the same Qwen3-Omni both to generate instructions and to accept results, so the two steps share the same model's biases — if Qwen3-Omni generates an instruction whose execution quality it itself finds hard to judge (e.g., some class of fine-grained attribute change), it may likewise make an incorrect judgment during acceptance, forming a self-consistent blind spot. This is a methodological weakness of this pipeline relative to Foley-Omni in terms of verification independence. Mitigating factors are that the evaluation set additionally goes through 20-person manual five-dimension review, and external AvED-Bench zero-shot verification.

[Design of the division of labor between human and machine verification] Automatic verification covers all 79K training samples (preserving scale), while manual verification is applied only to the 1K evaluation samples (preserving trustworthiness). The organization of the manual side is worth recording: 20 volunteers organized into 10 judge pairs, with each candidate sample assessed by 5 independent judge pairs separately, each pair responsible for only one of the five dimensions — i.e., "one judge pair looks at only one dimension" independent per-dimension review, avoiding the halo effect and cross-dimension contamination that occurs when a single person scores multiple dimensions at once. This design is relatively meticulous among peer works.

[Uncertain] Not disclosed: the scoring scale (binary pass/fail or continuous score + threshold), the pass threshold for each dimension, the prompt template, the actual elimination rate at the verification stage (this is key to quantifying the data engine's success rate, and it is missing), and any consistency check between Qwen3-Omni's judgments and human judgments (comparing the automatic and manual conclusions' agreement rate on the 1K manual set would have strongly corroborated the reliability of automatic verification, but this was not done).

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

Safety filtering exists explicitly in this pipeline as one of the five dimensions of automatic verification — "(v) safety to filter inappropriate content" — i.e., Qwen3-Omni judges whether the synthesized result contains inappropriate content, discarding it if not passed. Compared to most works in this survey that omit safety filtering entirely, this work at least writes it into the pipeline as one of the hard pass conditions — a moderately above-average practice.
[Limitations and gaps]
  · Coarse granularity: only a single generic "safety" dimension, with no breakdown into NSFW, violence, hateful content, copyright marks, facial privacy, etc. sub-categories, and no statement of what safety criteria or policy was used.
  · No dedicated safety models seen: no mention of NSFW classifiers, copyrighted-content recognition, face detection/de-identification, or other specialized tools — everything relies on the judgment of a general-purpose MLLM.
  · Applied late in the pipeline: the safety check only applies to synthetic products; no pre-screening for safety is seen on input material (YouTube-scraped content).
[The most notable gap — identity misuse risk unaddressed] The dataset includes three dedicated subsets — clone_id (identity cloning), clone_voice (voice-timbre cloning), clone_id_voice (identity + voice-timbre cloned simultaneously) — and the model's direct capability is "swapping faces, voices, and dialogue on real people's videos while maintaining lip sync" — precisely the technical definition of a deepfake. However:
  · The paper does not mention any licensing or informed consent for speakers' portrait rights/voice rights;
  · No watermark, C2PA content credential, or detectable marking is applied to generated content;
  · The paper's limitations discussion focuses entirely on technical shortcomings (physical realism, lighting consistency, 3D spatial consistency, object permanence), with no ethical statement or misuse-risk discussion — compared to Foley-Omni, which at least explicitly acknowledges deepfake risk, this work is even more lacking on this point;
  · The dataset is fully open for download under an MIT license (including real people's imagery), the weights are also fully open, and dedicated identity/voice-timbre cloning fine-tuning checkpoints are specifically provided, actually lowering the barrier to misuse.
Overall, the safety dimension is the aspect of this work most mismatched with its degree of technical openness.

## Annotation Method

### Captioning Model Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

This work does not perform traditional-sense video captioning (describing video content), but rather "editing instruction generation" (describing how to change the video); thus the "captioning model" here corresponds to an instruction-generation model along with a series of structured annotation tools.

[Instruction-generation model] Qwen3-Omni (Xu et al., 2025b, Alibaba Tongyi's open-source omni-modal large model). The rationale for this choice is native support for joint video+audio+text input, enabling simultaneous understanding of editable entities in the frame and the sound emitted by that entity in the audio track, thereby generating joint audio-video editing instructions. [Uncertain] The specific size tier used is not disclosed (Qwen3-Omni has multiple sizes), no prompt template is given, it is not stated how many instructions are generated per source, and it is not explained how instruction diversity is ensured (whether a template library, temperature sampling, or category-balanced sampling was used).

[The same model doubles as judge] Qwen3-Omni also performs the five-dimension verification scoring in Stage Three (see model_as_data_judge). Generation and acceptance sharing the same source is a methodological hazard in this pipeline.

[The accompanying structured annotation toolchain]
  · Grounded-SAM-2 (Ren et al., 2024) — open-vocabulary detection + segmentation + video tracking, producing instance-level masks, providing an anchor of "editable objects" for the instruction, and also defining the edit region for mask-guided synthesis.
  · TalkNet (Tao et al., 2021) — active speaker detection, annotating who is speaking and their spatiotemporal position.
  · ElevenLabs Scribe (2025) — ASR, extracting precise speech timestamps.
  · SAM-Audio (Shi et al., 2025) — the audio-side "segment anything," semantically separating the target entity's sound from the mixed audio track.
  · Semantic sound-event labels — assigning a unique label (e.g., "dog barking") to each non-speech clip; the labeling tool used is not specified.

[Assessment of tool choice] The entire chain uses exclusively open-source or commercial off-the-shelf models, with no in-house captioner, and no distillation or fine-tuning applied to Qwen3-Omni. The advantage is relatively high reproducibility (except that ElevenLabs is a commercial API) and low engineering cost; the disadvantage is that the capability ceiling of every step is entirely bounded by the chosen off-the-shelf models, and the paper does not conduct any comparative experiment on tool selection.

### Caption Density and Structuring Level (Short/Long/Dense Description, Structured Fields Such as Camera Motion, Style Tags) ⚠️

This work's "text condition" is an editing instruction rather than a content description, and its structural characteristics differ fundamentally from the dense captions of generation-type models — it describes a differential (what to change), not a state (what it is).

[Disclosure at the paper level] Only expressed as "diverse text instructions for comprehensive editing operations," covering the four editing operation types. [Uncertain] The paper gives no instruction template, average length, word-count statistics, grammatical structure, or any example statistics.

[The actual structure revealed by the HuggingFace dataset card (an important corroborating source not mentioned in the paper)] Each CSV record contains four fields:
  · original_video — source video path
  · target_video — target video path
  · instruction — forward editing instruction (source → target)
  · instruction_reverse — reverse editing instruction (target → source)
[The significance of bidirectional instructions] The instruction_reverse field is a design entirely unmentioned in the paper's main text but of considerable methodological value: for every synthesized (source, target) pair, two training samples are naturally obtained — "A plus a forward instruction yields B" and "B plus a reverse instruction yields A." This directly doubles the amount of data, and the "source" of the reverse sample is the synthetic product while the "target" is the real material — the opposite direction from the forward sample — which can mitigate the bias of the model only learning to imitate the data engine's output style (in forward samples, the target is always synthetic; if trained only on forward samples, the model's output distribution would be anchored to Wan2.2-5B's synthetic distribution; adding reverse samples means half the samples now target the real-video distribution). This is a clever, low-cost gain point in synthetic data construction. It also partially explains one source of the discrepancy between the HF-released 87K training pairs and the paper-reported 79K.
[Speech markers] Instructions may contain <S> and <E> markers to denote spoken content boundaries — i.e., a segment of target dialogue text wrapped in special tokens is embedded within the instruction, layering "what to say" (precise content) separately from "how to edit" (editing intent) within the same instruction. This is a lightweight form of structuring: the instruction body is natural language, while the spoken content is marker-wrapped literal text, preventing the model from misparsing the dialogue as editing intent.
[Comparison with visual-side structured labels] There are no structured fields common in text-to-video work such as camera motion, style tags, or camera parameters — because the visual context of the editing task is provided directly by the source video itself (entering the model via Source Concatenation), with no need for text as an intermediary description. This is a natural simplification of editing paradigms relative to generation paradigms in terms of text conditioning.
[Uncertain] The instruction length distribution, the linguistic expression patterns of the four editing operation types within instructions, whether an instruction template library exists, and what proportion of samples the <S>/<E> markers appear in — none of this is disclosed.

### Joint Audio-Video Caption Structure (Whether Visual and Audio Tracks Are Both Covered, Whether Split into Separate Fields, e.g., LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields)

This work's form of joint audio-video text conditioning differs markedly from other works in this survey: it is not a "joint caption" but a "joint instruction" — the object it describes is a cross-modal change operation, not cross-modal content.

[Whether both audio and visual tracks are covered] Covered, but through implicit unified coverage rather than explicit splitting. A single instruction implicitly entails both a visual change and an auditory change, and the two are bound by semantic causality — "replace the dog with a cat" requires, in one single instruction, that the dog in the frame become a cat AND that the barking on the audio track become meowing. The instruction itself is not split into two fields, a "visual instruction" and an "audio instruction"; instead, the model is expected to infer, from a single natural-language instruction, what change each modality should undergo. This is the core premise of this work: the paper emphasizes that joint editing can be accomplished "given solely text instructions," with the user not needing to separately specify how audio and video should each change.

[Comparison with other schemas in this survey's lineage]
  · LTX-2: unified full-soundscape description (a single passage of text describing the entire soundscape) — descriptive type, single audio track.
  · Script-a-Video: factorized streams (decomposed into independent visual and auditory streams) — descriptive type, dual-track split.
  · Foley-Omni: [WORDS]/[AUDIO]/[MUSIC] three fields — descriptive type, three-way auditory split.
  · InstructAV2AV: a single natural-language editing instruction + <S>/<E> speech-content markers — instructive type, modality-fused and unsplit.
  It can be seen that this work sits at the lowest end of the "degree of structuring" axis (almost no field-ization), but this is not a design shortcoming — it is a consequence of the task's nature: if the editing instruction were forcibly split into "what to change visually / what to change aurally," it would undermine its core selling point (the user just needs to say one sentence in plain language) and introduce the risk of the two modal instructions being inconsistent. The benefits of structuring (independent control, independent verification) give way to usability and cross-modal consistency in the editing scenario.

[Traces of structuring still present in two places]
  1. The <S>...<E> markers — layering the spoken literal content (the new dialogue to be said) apart from the editing intent, the only explicit field-ized design.
  2. The instruction / instruction_reverse bidirectional fields — explicitly structuring the editing direction.

[No text description on the visual side] The content of the source video does not enter the condition as text, but is injected directly into the diffusion process via Source Concatenation (source latent concatenated with noise latent). Thus the model's "visual understanding" travels through the latent pathway rather than the text pathway, consistent with Foley-Omni's approach of carrying visual information via CLIP/Synchformer features — both are cases of "visual information bypassing text as an intermediary."

[The joint-ness guarantee on the data-construction side] Worth noting is that audio-video joint consistency is guaranteed mainly not by the schema, but by the generation order of the data engine: the new audio track is synthesized first according to the instruction, then the new footage is synthesized conditioned on the audio features via frame-wise cross-attention. That is, "jointness" is enforced within the causal chain of data generation, rather than declared within the annotation structure. This is a distinctive feature of this work's path to achieving jointness compared to other works.

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion) ⚠️

Dialogue processing is the most complex of the four task types in this work (corresponding to HF's clone_id / clone_voice / clone_id_voice subsets), with a complete processing chain but insufficient disclosure of attribute annotation.

[Transcription step] Uses ElevenLabs Scribe (2025) for ASR, with the explicitly stated purpose being to "extract precise speech timestamps" — note the emphasis on precise timestamps rather than mere text content. Timestamp precision is a hard requirement for this task: the edited new dialogue must replace content within the original speech's time window, and the new lip movements must be aligned frame by frame with the new speech — inaccurate time boundaries would directly cause lip-sync failure.

[Speaker localization and on-screen determination] TalkNet (Tao et al., 2021) performs active speaker detection, locating "the person who is currently speaking" in the frame. Combined with the transcription result, a hard admission condition is applied: a clip is retained only when speech is temporally aligned with a visible on-screen speaker (clips retain speech only when temporally aligned with a visible on-screen speaker). This single rule simultaneously excludes voiceover, narration, commentary, post-production dubbing, and scenes where the speaker is off-screen, equivalent to enforcing at the data level that "all speech samples inherently come with lip-sync supervision." This is more thorough than post-hoc filtering with a SyncNet score — the latter filters out synchronized samples from a mixed pool, whereas the former only admits sound-and-image co-sourced samples in the first place.

[Decoupled annotation of speech synthesis and identity/timbre] The naming of HF's three subsets reveals an implicit attribute-dimension system:
  · clone_id — swaps visual identity, preserves the original voice-timbre;
  · clone_voice — swaps voice-timbre, preserves the original visual identity;
  · clone_id_voice — swaps both visual identity and voice-timbre simultaneously.
  This means "visual identity" and "auditory timbre" are treated as two independently controllable orthogonal factors during data construction, with controlled samples constructed for three combinations respectively. This is a capability unique to synthetic data: in reality it is impossible to capture genuine material of "the same video with the face swapped but the voice unchanged" — only controlled synthesis can provide this kind of disentangled supervision, letting the model learn to control identity and timbre separately. This is the most persuasive example in this work of the "representativeness of the synthetic data dimension."
  Speech synthesis is performed by ElevenLabs (a platform known for its voice-cloning capability); it is inferred that clone_voice-type samples use its voice-cloning function to generate a new timbre, while identity-preserving speech modification-type samples use its capability to preserve the original timbre while only changing the content — but the paper does not describe the specific configuration.

[Uncertain] A large amount of attribute annotation information is missing: no structured annotation fields are seen for speaker identity ID, gender, age, language, accent, or emotion; it is not stated whether the ASR transcription text is retained as a data field (judging from the <S>/<E> markers in the instruction, the target dialogue text does enter the instruction, but whether the source transcription is retained is unknown); the ElevenLabs invocation configuration (voice-cloning parameters, speech-rate control, emotion settings) is not described; no quantitative evaluation of transcription quality or synthesized speech quality is performed. The evaluation side uses Sync-C / Sync-D to measure lip sync, but does not use WER to measure speech content accuracy — meaning the most direct fidelity metric for speech editing, "did the model actually say the dialogue required by the instruction," is missing from evaluation.

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Annotation, Explicit State) ⚠️

This work uses two types of structured visual annotation, but both are instrumental intermediate products of the editing task rather than annotations retained for training.
[Instance-level segmentation masks] Grounded-SAM-2 (Ren et al., 2024) produces instance-level masks, the most important structured annotation in this pipeline. It has a dual purpose: (1) providing an anchor of editable objects for Qwen3-Omni's instruction generation ("which entities in the frame can be referenced by an instruction"); (2) defining the edit region for the mask-guided video editing model, structurally guaranteeing that pixels outside the mask are not altered — a harder mechanism for "content preservation" at the data level than relying on the model's own self-discipline to leave the background alone. Note that the mask is used only during the data synthesis stage; the final model InstructAV2AV is mask-free (the user only needs to provide a text instruction, no mask), meaning the mask's information is implicitly distilled into the model through the data. This "use a mask during construction, no mask at inference" design is a clear value point of this work.
[Point trajectories] CoTracker3 performs grid-based point tracking, producing point trajectories used to compute average motion magnitude. The trajectories themselves are used only as a filtering criterion, not retained as annotation.
[Speaker spatiotemporal localization] TalkNet outputs the active speaker's spatiotemporal position, used for sound-image attribution judgment, likewise for filtering purposes only.
[Uncertain] No 3D- or camera-level annotation is involved at all: no camera intrinsics/extrinsics, no camera trajectories, no depth maps, no 3D point tracks, no human/facial keypoints or pose annotation, no explicit object-state annotation.
[This gap's direct connection to the paper's limitations] The paper acknowledges that the model "has difficulty preserving 3D spatial consistency or object permanence during extensive camera movements." This failure mode is directly related to the data side's lack of camera and depth annotation, and the lack of any layered control over camera motion — the data neither tells the model how the camera is moving, nor applies any ratio control or graded curriculum across different camera-motion intensities, so the model can only learn implicitly from pixels, and naturally fails under large motion. If future work introduces camera-parameter annotation or buckets samples by camera-motion intensity, that is a clear improvement path.

### Synthetic Data Construction (Controlled Perturbation/Editing to Construct Training Pairs, e.g., InstructAV2AV) ⚠️

This is the most representative dimension of InstructAV2AV in this survey, and the reason the note lists it as the "representative of the synthetic data dimension." In this work's training data, every single target is a synthetic product — not genuinely filmed, but manufactured under control.

[Why synthesis is essential: the fundamental predicament of the task] Instruction-guided editing requires (source, instruction, target) triple supervision, where source and target must be two videos that are "the same scene, differing only in the target region." Such pairs simply do not exist in reality — you cannot film the same scene twice with only a dog turning into a cat and every other pixel remaining exactly the same. The paper explicitly starts from this premise: "Given the scarcity of such source-to-target paired resources in the audio-visual domain." Therefore, data synthesis is not an efficiency measure but the precondition for the task to be viable at all.

[Three layers of controlled design in the synthesis mechanism]
  1. Spatially controlled (visual side): Grounded-SAM-2 produces instance masks, and the mask-guided editing model (based on Wan2.2-5B, trained with a source-conditioned flow-matching objective) synthesizes new content only within the mask region, with pixels outside the mask coming from the source video. This structurally guarantees that "non-target regions remain strictly unchanged," making the source-target pair a pixel-level clean differential pair.
  2. Sound-source controlled (audio side): SAM-Audio separates the target entity's sound from the original mixed audio track, ElevenLabs synthesizes the new target sound, and it is then seamlessly mixed with the retained original background sound. Likewise this guarantees that "non-target sound remains strictly unchanged," with ambient/background sound completely consistent between the two sides.
  3. Cross-modal synchronization controlled: the video synthesis model receives the already-synthesized target audio features as a condition via frame-wise cross-attention, forcing the new footage to align frame by frame with the new audio track. The synthesis order is "audio first, video second" — once the duration and rhythm of the speech are fixed, lip movements must follow it.
  These three layers of control together produce something real data can never provide: precise, factor-level differential supervision.

[Orthogonal-factor decoupled construction] The five subsets add_and_remove / clone_id / clone_voice / clone_id_voice / general_editing embody independent control and combinatorial enumeration over a few factors — "visual identity," "auditory timbre," "entity presence or absence." In particular, the pair clone_id (swap face, not voice) and clone_voice (swap voice, not face) constitutes a strict control-group pair — the model can learn from these that identity and timbre are separable attributes. This kind of decoupled supervision is a core advantage of synthetic data over real data, and the most valuable construction idea of this work worth borrowing.

[The data gain from bidirectional pairing] The HF dataset includes an instruction_reverse field, so a single (source, target) pair can be used as both a forward (real→synthetic) and reverse (synthetic→real) training sample. The generation target of the reverse sample is the real video, which can mitigate the problem of the model's output distribution being anchored to the synthetic distribution.

[Inherent risks of the synthetic route (undiscussed by the paper)]
  · Inherited capability ceiling: the quality ceiling of the target is determined by Wan2.2-5B and ElevenLabs; the editing effects InstructAV2AV learns cannot, in principle, exceed its data engine. The limitations acknowledged by the paper (physical realism, lighting consistency, 3D consistency) are explicitly stated as being "inherit[ed] from the foundational generation model" — precisely a manifestation of this risk.
  · Self-reinforcing distributional bias: the training set and evaluation set (1K) come from the same engine, so the model might score well by "imitating this engine's style" rather than by truly being good at editing. The paper's use of external zero-shot evaluation on AvED-Bench partially addresses this concern (FVD 227.82 vs. AVI-Edit's 372.37).
  · Same-source failure modes: systematic flaws in the data engine (e.g., a certain class of object consistently synthesizing poorly) are fully transmitted to the downstream model and cannot be caught by automatic verification — because the Qwen3-Omni used for verification and the Qwen3-Omni used for generation are the same model.
[Uncertain] The one-pass success rate of the data engine, the average number of paired samples produced per source, and the compute cost and total time of synthesis are not disclosed.

### Degree of Human Involvement (Manual Annotation, Manual Quality Inspection, Model Pre-Screening + Manual Review) ⚠️

The degree of human involvement is low and highly focused, concentrated in only two places — evaluation-set gatekeeping and the user study — with the construction and acceptance of the 79K training data being fully automatic.

[Manual acceptance of the evaluation set (the only step involving human participation in constructing training/evaluation data)] The organization is disclosed in considerable detail: 20 volunteers organized into 10 judge pairs, with each candidate sample assessed by 5 independent judge pairs, each pair responsible for only one of the five criteria; a sample must pass all five to enter the 1K evaluation set.
  Two things about this design are worth recording: (1) the "one judge pair looks at only one dimension" independent per-dimension review avoids the halo effect and cross-dimension contamination that occurs when a single person scores all dimensions, more rigorous than the common practice of "one person scores all dimensions"; (2) using a "pair" rather than a single individual as the review unit implicitly provides internal-consistency cross-checking.
  [Uncertain] It is not stated how disagreement within a judge pair is resolved, no inter-rater agreement (Cohen's kappa) is reported, the volunteers' backgrounds and training method are not described, and it is not reported how many candidates were filtered down to reach the 1K.

[User study (an evaluation activity, not part of data construction)] 25 volunteers, with 20 samples randomly drawn from each dataset, made preference choices along three dimensions: audio-video synchronization (AVS), text alignment (TA), and overall preference (OP). Results: on InsAVE-80K, AVS 49.00% / TA 45.40% / OP 46.60%; on AvED-Bench, AVS 46.80% / TA 41.80% / OP 43.60% — the highest preference rate in each four-way comparison (if the four methods were tied, each would get about 25%, so 40%+ is a significant lead). [Uncertain] Volunteer recruitment method, whether there was overlap with the 20 people who did evaluation-set acceptance, scoring rubric, and consistency checks are not described.

[Zero human involvement in training data] The 79K training pairs are entirely accepted automatically by Qwen3-Omni, with no manual review, no sampling-based quality inspection, and no report of the agreement rate between automatic acceptance and human judgment. This is the price of scalability, and also the inevitable choice given the positioning of "scalable data synthesis pipeline" — but the absence of a cross-validation experiment "comparing automatic and manual conclusions on the 1K manual set" means the reliability of automatic acceptance cannot be quantified — a regrettable methodological gap.

[The role of human input in threshold calibration] [Uncertain] The basis for determining the CoTracker3 motion threshold, LAION aesthetic threshold, Audiobox threshold, and MLLM scoring threshold is not described; it is presumed there is some empirical manual-tuning component, but the paper does not state this explicitly the way Foley-Omni does with "determined by manually inspecting a small-scale validation subset."

## Audio-Video Alignment

### Audio-Video Sync Detection Method (Lip Sync, Event Alignment) ⚠️

Audio-video synchronization is handled in this work as a problem across three different stages — data admission, enforcement at synthesis time, and result evaluation — with different mechanisms at each of the three, making this the most layered part of sync handling in this pipeline.

[Step one: sync assurance at the data admission stage (upstream control, not score filtering)]
  Rather than using a sync scorer such as SyncNet/Synchformer for threshold filtering, a stronger admission condition of "sound-image attribution" is used: TalkNet locates the active speaker + Scribe extracts speech timestamps, retaining only clips where "speech is temporally aligned with a visible on-screen speaker." On the non-speech side, a single clear sound source is required (removing ambiguous sound sources). This is effectively admitting only sound-and-image co-sourced material from the outset, rather than filtering synchronized samples out of a mixed pool — more thorough than score-threshold filtering, because it excludes structural problems like "the sound doesn't come from the frame at all," not merely time offsets.

[Step two: sync enforcement at the data synthesis stage (guaranteed at generation time)]
  This is the most distinctive point of this work: synchronization is not filtered out, but enforced at generation time. The mask-guided video editing model injects the already-synthesized target audio features frame by frame into the video generation process via frame-wise cross-attention, phrased by the paper as "ensuring strict temporal synchronization." The synthesis order is audio-first, video-second, letting the lip movements/actions follow the established audio timeline. Compared to "generate each independently then filter out the unsynchronized ones," this is far more efficient — unqualified samples are simply never generated in the first place.

[Step three: sync re-checking at the automatic verification stage]
  The fourth of Qwen3-Omni's five-dimension scoring criteria is "audio-video synchronization for cross-modal alignment," used as one of the hard pass conditions, performing a semantic-level sync review of the synthesized result. This uses an MLLM rather than a dedicated sync model for judgment — the advantage is capturing semantic-level mismatches (e.g., cat visuals paired with dog barking), the disadvantage being lower time precision than dedicated models like SyncNet.

[Step four: sync measurement at the model evaluation stage]
  Four audio-video metrics are used: AV-A (ImageBind-based audio-video semantic alignment), PEAVS (Perceptual Evaluation of Audio-Visual Synchrony), Sync-C (SyncNet confidence, higher is better), and Sync-D (SyncNet distance, lower is better, configured with vshift=15 frames). InstructAV2AV scores AV-A 27.72 on the InsAVE-80K evaluation set, better than AvED's 26.44, AVI-Edit's 26.37, and CoherentAVEdit's 22.67; on AvED-Bench, AV-A 23.71 vs. AVI-Edit's 23.21.

[Methodological assessment] The four-layer design of "upstream admission + generation-time enforcement + MLLM re-checking + evaluation-time measurement" is more complete than relying purely on a sync-score threshold, with clear responsibilities at each layer (structural problems solved at the admission layer, temporal precision guaranteed at the generation layer, semantic matching gatekept at the verification layer, and final capability measured at the evaluation layer).
[Uncertain] No dedicated sync-detection model (SyncNet/Synchformer/AV-HuBERT) is used anywhere in the data-construction process for quantitative filtering, so no sync-quality distribution statistics can be given for the dataset itself; nor is any ablation on "with/without frame-wise cross-attention" performed to quantify the mechanism's contribution to the sync quality of the synthesized data.

### Specific Sync Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

[Data-construction side: no numerical thresholds]
  [Uncertain] This is the most obvious difference between InstructAV2AV and works such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 and conf>1.5), and Foley-Omni (Synchformer≥0.2, ImageBind≥0.3) — InstructAV2AV sets no numerical sync-score threshold whatsoever during data construction. Its sync quality is guaranteed by two non-threshold mechanisms instead: (1) TalkNet+Scribe's boolean admission condition of "speech must be temporally aligned with a visible on-screen speaker"; (2) the architecture-level enforcement of frame-wise cross-attention at the synthesis stage. This is a "replace thresholds with mechanisms" approach — the benefit is not needing to calibrate a threshold and not risking false eliminations or leaks due to a poorly set threshold; the drawback is that the dataset's sync quality cannot be quantitatively described or compared horizontally with other datasets.
  The only data-side threshold with an exact value is the -45 dBFS audio silence threshold (PyDub), but that is not a sync metric. The remaining thresholds (CoTracker3 motion magnitude, LAION aesthetics, Audiobox-Aesthetics, MLLM five-dimension scoring threshold) are all not given numerically.

[Evaluation side: metrics and configuration are explicit]
  · Sync-C (S-C, higher is better) — SyncNet lip-sync confidence.
  · Sync-D (S-D, lower is better) — SyncNet sync distance/offset. Both are explicitly configured with vshift=15 frames (i.e., searching for the best alignment position within a ±15-frame window, SyncNet's standard setting).
  · PEAVS (higher is better) — perceptual-level audio-video sync evaluation metric, unlike SyncNet not limited to lips, covering general audio-video event synchronization, a necessary complement for this work's non-speech editing tasks (instance insertion/removal).
  · AV-A (higher is better) — ImageBind-based audio-video semantic alignment score. On the InsAVE-80K evaluation set, InstructAV2AV scores 27.72 vs. AvED 26.44 / AVI-Edit 26.37 / CoherentAVEdit 22.67; on AvED-Bench, 23.71 vs. AVI-Edit 23.21.
  The four metrics cover three levels: "lip-sync precision (Sync-C/D)," "general event-sync perception (PEAVS)," and "cross-modal semantic matching (AV-A)" — a fairly complete breakdown.
  [Uncertain] The specific values of Sync-C / Sync-D / PEAVS are not fully presented in the accessible table content; only AV-A has exact figures.

[Methodological gap] No threshold on the data side, but metrics on the evaluation side, mean there is no common yardstick to cross-reference "the data's sync quality" against "the model's sync capability." If Sync-C/Sync-D had also been measured and reported for the dataset's sync-score distribution during data construction, the key question of "has the model's sync capability already approached the data ceiling" could have been answered — unfortunately this was not done.

### Separation of Temporal Sync vs. Semantic Sync (Temporal Alignment and Content-Semantic Matching Treated as Two Independent Filtering Conditions) ⚠️

This work performs a substantive separation of temporal synchronization and semantic matching, but the location of this separation differs from Foley-Omni — Foley-Omni places two parallel thresholds within the same filtering stage, whereas this work dispatches the two concerns to different steps of the pipeline, each handled by a different mechanism.

[Temporal sync handled by "structural mechanisms"]
  · Data admission layer: TalkNet + Scribe's "speech must be temporally aligned with a visible on-screen speaker" is a purely temporal criterion.
  · Data synthesis layer: frame-wise cross-attention injects audio features frame by frame into video generation — architecture-level temporal enforcement.
  · Evaluation layer: Sync-C / Sync-D (vshift=15), PEAVS measure temporal alignment precision.

[Semantic matching handled by "large-model judgment"]
  · Data admission layer: non-speech clips are required to have a "single clear dominant sound source" and are assigned a semantic sound-event label (e.g., dog barking), effectively confirming the semantic attribution of the sound source.
  · Instruction-generation layer: Qwen3-Omni generates instructions conditioned on instance masks and audio-video context, naturally requiring that "this visual entity" and "this sound" be semantically bound (dog ↔ barking); otherwise the generated joint editing instruction is not coherent.
  · Automatic verification layer: the "audio-video synchronization for cross-modal alignment" dimension among the five, judged by the MLLM, actually leans more toward semantic matching (is the frame a cat and the audio track a meow) than frame-level temporal precision.
  · Evaluation layer: AV-A (ImageBind cross-modal semantic similarity) measures semantic matching.

[Why this dispatch is reasonable] The nature of the two problems determines which tool applies: temporal alignment is signal-level, precisely measurable, and can be enforced architecturally at generation time; semantic matching is content-level, requires understanding, and can only be judged by a model. This work hands signal problems to signal mechanisms and semantic problems to large models, with a clear division of labor. By comparison, Foley-Omni's use of two scoring models (Synchformer for temporal, ImageBind for semantic) in parallel filtering is the same idea implemented under a pure-filtering paradigm.

[An implicit advantage] Since the target audio is synthesized first and the video is then synthesized conditioned on it, temporal alignment in the data is naturally near-perfect (the audio-visual frame-misalignment problems common in real material simply do not occur), giving the model an exceptionally clean temporal supervision signal during training. This also explains why this work dares to set no sync-score threshold at all on the data side.
[Uncertain] No ablation is done on the necessity of separating the two, nor is any independent quality statistic reported for semantic matching vs. temporal alignment within the dataset.

### Audio Quality Filtering (SNR, Silence Detection and Silence-Ratio Thresholds, Removal of Tracks with No Audio, Removal of Off-Screen Sound Sources, Background Music Separation) ⚠️

Audio quality control is distributed across two levels: admission filtering and synthesis protection.

[Admission filtering (applied to real material)]
  · Silence detection: PyDub removes silent clips below -45 dBFS. This is the only threshold in the paper given an exact value. -45 dBFS is a relatively lenient bar (roughly "almost completely silent"), intended to remove clips with no audio track or pure silence, rather than fine-grained loudness control. [Uncertain] It is not stated whether this is measured as the whole-clip average level or determined segment by segment, and no upper limit on silence proportion is given (such as a constraint like "remove if silent duration exceeds 30%").
  · Audio quality assessment: Audiobox-Aesthetics (Tjandra et al., 2025, Meta's reference-free audio aesthetics evaluation model) removes low-quality audio. This is the same tool used by Foley-Omni, part of the 2025–2026 new paradigm of audio-data cleaning (using learned perceptual quality assessment to replace traditional signal metrics like SNR/THD, able to identify issues traditional metrics cannot detect, such as over-compression, clipping, and encoding degradation). [Uncertain] The threshold value is not given (Foley-Omni's corresponding threshold of ≥0.6 can serve as a reference scale).
  · Sound-source clarity: non-speech clips with ambiguous sound sources are removed, requiring that a single dominant sound source be clearly identifiable. This is a filtering condition serving downstream separability, uncommon in general-purpose audio cleaning.
  · Off-screen sound-source removal: speech clips are required to have "speech temporally aligned with a visible on-screen speaker," effectively removing narration, commentary, voiceover, and post-production dubbing — a hard admission condition in this work rather than a soft filter, more thorough than in most works.

[Handling of background music/ambient sound — protection rather than removal]
  This is an important difference from most V2A works: background sound is not interference to be removed, but a "content preservation" supervision signal to be precisely retained. SAM-Audio (Shi et al., 2025) semantically separates the target entity's sound from the original mixed audio track, and after editing, the new sound is seamlessly mixed back with the retained background sound. Thus the background-sound components of source and target remain strictly consistent, constituting the auditory-side differential control benchmark.
  [Uncertain] It is not stated how residuals/artifacts from SAM-Audio's separation are handled — semantic source separation in practice often leaves spectral gaps or separation residue; if the target's background sound has subtle distortion from the separate-remix process, the model would learn this distortion as "normal." The paper does not discuss this, nor is there any verification or objective metric for separation quality.

[SNR] [Uncertain] No signal-to-noise ratio (SNR) threshold or any traditional signal-level audio metric is mentioned; everything relies on the learned, holistic Audiobox-Aesthetics evaluation.

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

Audio is split into a "speech vs. non-speech sound-source event" binary, with the two categories running through entirely independent processing chains; this branch point runs through the admission filtering, instruction generation, and audio synthesis stages, the clearest classification-driven design in the pipeline.

[Classification determination and basis] The paper does not explicitly state which classifier performs the speech/non-speech distinction; inferring from the toolchain, it is implicitly determined by TalkNet's (active speaker detection) output — an on-screen speaker detected routes to the speech path, otherwise to the event path. [Uncertain] The specific implementation of the classification and the sample ratio between the two categories are not disclosed.

[The speech-path processing chain]
  Admission: TalkNet locates the active speaker → Scribe extracts precise speech timestamps → the hard condition "speech must be temporally aligned with a visible on-screen speaker," otherwise discarded.
  Instruction: Qwen3-Omni generates speech-editing instructions, with the target dialogue wrapped in <S>...<E> markers embedded in the instruction.
  Synthesis: ElevenLabs speech synthesis, divided by subset type into modes such as preserving the original timbre while changing content (identity-preserving speech modification), and cloning a new timbre (clone_voice).
  Evaluation: Sync-C / Sync-D (SyncNet lip sync) measures this specifically.

[The non-speech event-path processing chain]
  Admission: clips with ambiguous sound sources are removed → each remaining clip is assigned a unique semantic sound-event label (e.g., "dog barking") based on its dominant sound source.
  Instruction: Qwen3-Omni generates entity editing/insertion/removal instructions based on instance masks and sound-event labels.
  Synthesis: SAM-Audio separates the target entity's sound → ElevenLabs text-to-audio synthesizes the new sound → mixed with the background sound.
  Evaluation: PEAVS (general event sync) and AV-A (cross-modal semantics) measure this specifically.

[The absence of music] [Uncertain] The paper does not list music as a separate processing category. This contrasts with Foley-Omni's speech/effects/music three-way split. It is presumed that music (the film material in the sources typically carries a score) is subsumed into the "background sound" separated by SAM-Audio and retained as-is — i.e., music is a protected background rather than an editable object — consistent with the task definition (there is no editing task like "change the score from sad to upbeat"). But the paper does not state this explicitly, nor does it discuss whether the presence of a score interferes with SAM-Audio's separation of the target entity's sound.

[Assessment of the binary vs. three-way split trade-off] This work's binary split is task-oriented: an editable object can only be "a talking person" or "a sounding object" — music has no corresponding visual entity that a mask can locate, so it is naturally outside the editing scope. This split is simpler than Foley-Omni's three-way split, but is sufficient for this task.

## Training Integration

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (Basis for Stage Division: Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long) ⚠️

Uses a Two-Stage Training Strategy (TSTS), with the basis for division being [modality coupling degree] — first adapting each modality separately for the editing task, then jointly fine-tuning, with the goal of smoothly transferring the pretrained prior to the editing task.

[Stage 1: Modality-specific training] The video branch and audio branch are optimized independently, without interfering with each other. Training steps are explicitly given: video branch 320K steps, audio branch 960K steps. The audio branch's step count is 3x the video branch's — a disparity worth noting — combined with the joint-stage loss weighting λ_v=0.85 / λ_a=0.15 (visual weight about 5.7x the audio weight), it can be inferred that the authors judged: the audio branch needs more steps to converge (hence 3x more training in Stage 1), but in the joint stage the audio's gradient needs to be suppressed to avoid interfering with vision (hence a weight of only 0.15). That is, a strategy of "let audio train thoroughly alone first, then speak less when joint."

[Stage 2: Audio-video joint training] The unified architecture is fine-tuned as a whole for 400K steps, with the two branches coupled through Ovi's twin-tower interaction mechanism, with a loss of λ_v·L_v + λ_a·L_a, λ_v=0.85, λ_a=0.15 (termed by the paper as empirical modality-balancing weights).

[Quantitative evidence of curriculum effectiveness] In the ablation, "W/o Two-Stage Training Strategy" (directly single-stage joint training) shows: FVD worsening from 180.38 to 291.55 (a 61.6% degradation), TV-A dropping from 25.23 to 24.87, FAD rising from 2.75 to 5.18 (an 88.4% degradation). The paper describes its failure as visual distortion and inconsistency. Note that the audio metric FAD's degradation (88%) is greater than the visual metric FVD's (62%), indicating the audio branch is more sensitive to curriculum design — consistent with Foley-Omni's finding (in that work, removing the three-stage curriculum worsened WER by nearly 4x, with speech suffering the most damage), possibly reflecting a cross-work commonality: in audio-video joint training, the more structurally rigid, less error-tolerant modality (audio/speech) relies more heavily on upstream single-modality pretraining, and its gradient is easily overwhelmed by the visual task under direct mixed training.

[Difference from common paradigms in the basis for stage division] This work's stage division is not based on resolution, duration, or data quality tiers (there is no "low-res→high-res," "short→long," or "coarse-filter→curated" curriculum), because the data specification is uniformly fixed at 5s/720p/24FPS/16kHz, with no gradable dimension. This is a side effect of a synthetic dataset: uniform specification brings engineering simplification, but also forecloses the possibility of a resolution/duration curriculum.

[Hyperparameters and hardware] 8x NVIDIA H100, AdamW optimizer, learning rate 1×10⁻⁵. [Uncertain] Batch size, total training time, warmup/scheduling strategy, and EMA settings are not disclosed. The three stages together total 1.68M steps on 8x H100 — a fairly substantial training volume, with actual wall-clock time likely on the order of weeks, but the paper does not report this.

### Changes in Data Mixture Across Training Stages (Pretraining/Annealing/SFT High-Quality Subset) ⚠️

[Uncertain] The paper does not disclose any data-mixture information — the main gap in this work regarding training-data integration.

[Known structural information]
  · Stage 1's two branches process different modality slices of the same batch of InsAVE-80K data — the video branch uses only video pairs, the audio branch only audio pairs — so there is no cross-source mixing issue, only a modality-dimension split.
  · Stage 2 uses the complete audio-video paired data.
  · Throughout, there is only one data source, InsAVE-80K (79K/87K pairs), with no multi-dataset mixing, annealing, or staged data-source switching design.

[The five subsets revealed by the HF dataset card, but no mixture ratio] The sample counts of the five categories — add_and_remove, clone_id, clone_id_voice, clone_voice, general_editing — how they were sampled during training (natural ratio or weighted balancing), and whether upsampling was applied to scarce categories — none of this is described by the paper or the data card. This is a rather regrettable omission: the difficulty and data volume of the five task types likely differ substantially (e.g., clone_id_voice, requiring simultaneous face and voice swapping, likely has the highest synthesis difficulty and lowest success rate, and very likely has significantly fewer samples than general_editing); without mixture-ratio control, the model's capability on scarce tasks would be noticeably weaker than on advantaged tasks — and the paper's evaluation is done on a mixed evaluation set with an overall metric, unable to reveal per-task capability differences.

[Modality weighting as a substitute for data mixture] The only explicit "mixture" is the loss-level modality-balancing weights λ_v=0.85 / λ_a=0.15, and the Stage 1 step allocation of video 320K / audio 960K (1:3). Together these two constitute this work's actual "mixture strategy" — not adjusting ratios at the data level, but adjusting weights at the gradient level and budget at the step level. This is a reasonable substitute for a scenario with a single data source.

[No annealing/SFT curation stage] No design of using a small high-quality subset for final annealing or SFT is seen. The 1K manually curated set is explicitly used for evaluation, not training.
[Uncertain] The internal sampling strategy of each stage, the usage ratio of forward vs. reverse instruction (instruction / instruction_reverse) samples, and the sampling weights of the five subsets are not disclosed.

### Post-Training Data (SFT Curated-Set Scale and Selection Criteria, Preference-Pair Count and Annotation Method, Reward-Model Training Data) ⚠️

This work's training pipeline stops at two-stage supervised training (with Stage 2 joint fine-tuning as the final stage), with no independent preference-alignment post-training step.

[Overall positioning clarification] It's worth noting that InstructAV2AV itself is by nature closer to "post-training": its generation capability comes from upstream Ovi's pretrained prior, and InsAVE-80K's 79K editing pairs play the role of an instruction-tuning dataset — teaching an already-existing audio-video generation model "how to perform localized editing according to instructions." From this angle, the entire InsAVE-80K is this work's post-training data, and its scale (79K pairs / about 110 hours) is also consistent with the typical scale of instruction-tuning sets.

[The filtering criteria are the quality standard for the post-training data] Samples entering the training set must pass all five of Qwen3-Omni's five-dimension automatic verification criteria (instruction fidelity / content preservation / perceptual quality / audio-video synchronization / safety).

[Preference data and RM] [Uncertain] No preference pairs were constructed, no reward model was trained, and no DPO/PPO/GRPO-type preference optimization was used. The preference data collected in the user study (25 people × 20 samples × 3 dimensions) was used only for final evaluation reporting, not fed back as a training signal — this data could have supported a round of DPO if used to construct preference pairs, but the paper did not do this.

[Use of the manually curated set] The 1K samples with triple manual gatekeeping (20 volunteers / 10 judge pairs / independent per-dimension review) are explicitly used for the evaluation set, not for training. This is reasonable (using them for evaluation is the way to reflect their gatekeeping value), but it also means the training side has no manually-verified samples whatsoever.

[Gap relative to industrial models] Compared to industrial models like Veo 3 and Sora 2, which commonly include a preference-alignment stage, this work is blank on the post-training dimension. This may be one reason why, despite leading in subjective preference rate (OP 46.60%), it does not win overwhelmingly — pure supervised fine-tuning struggles to align with subjective judgments of "which edited result humans find better."

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Acceleration Ratio, Processing Scale, Cost) ⚠️

[Uncertain] The paper discloses no data-processing infrastructure information whatsoever: no mention of NeMo Curator, Data-Juicer, or an in-house distributed processing framework, no processing throughput (clips/GPU-day), cluster scale, total time, or cost accounting is given. Only the model-training-side hardware (8×NVIDIA H100) is disclosed; it is not stated how much compute was used for data synthesis.

[An inferable cost structure (this work's cost profile differs fundamentally from pure cleaning pipelines)]
  A pure cleaning pipeline's cost is mainly "filtering" (lightweight-model batch inference), whereas this work's main cost is "generation" — every single training sample requires a full diffusion-model sampling run, an order of magnitude more expensive:
  · Most expensive: synthesis by the mask-guided video editing model (based on Wan2.2-5B). To produce 79K 5-second/720p/24FPS target videos, at least 79K runs of diffusion sampling are required (accounting for failed retries, this could reach 150K–400K per the earlier extrapolation). A 5B model generating a 5-second 720p video typically takes tens of seconds to a few minutes per run on an H100, cumulatively reaching the tens-of-thousands-of-GPU-hours scale. This is the single largest cost item in the whole pipeline.
  · Second most expensive: ElevenLabs commercial API calls (speech/audio synthesis), billed per call, with 79K+ calls representing a considerable expense.
  · Moderate: Qwen3-Omni's two rounds of invocation (instruction generation + five-dimension verification); inference cost for an omni-modal large model on video+audio is not low, and the verification round must cover all candidate samples (including those eliminated).
  · Lighter: PySceneDetect, CoTracker3, LAION Aesthetics, PyDub, Audiobox-Aesthetics, TalkNet, Grounded-SAM-2, SAM-Audio, and other upstream filtering and annotation models, all small-to-medium models capable of batch inference.
  · Commercial API dependency: ElevenLabs (Scribe ASR + TTS/TTA) is the only closed-source commercial dependency, constituting a reproducibility obstacle and an uncontrollable cost point.

[Self-positioned scalability vs. the evidence gap] The paper positions itself with "scalable data synthesis pipeline" as one of its core selling points, but provides no engineering data whatsoever to support this scalability claim — no throughput, no per-sample cost, no parallelization scheme, no failure-retry strategy, and no synthesis success rate. For a pipeline claiming to be "scalable," these are exactly the metrics that should have been reported. On the open-source side, GitHub provides inference and pipeline scripts, but training scripts have not yet been released, and it is unclear whether the complete data-synthesis-engine code is included in the released pipeline scripts.

## Results Comparison

### Quantitative Impact of Data-Strategy Ablations (Distinguishing: Filtering-Strictness Ablation / Caption-Density-Style Ablation / Data-Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

The paper's ablation experiments focus entirely on model architecture and training strategy, with no data-strategy ablation performed whatsoever — the most obvious gap in this work's experimental design, especially given that it lists "scalable data synthesis pipeline + InsAVE-80K" as two of its three major contributions.

[Ablations performed (in table form, on the InsAVE-80K evaluation set)]
  Full-model baseline: FVD 180.38 / TV-A 25.23 / TC 95.65 / FAD 2.75 / TA-A 34.96 / AV-A 27.72
  · W/o Source Concatenation (removing source concatenation): FVD 467.20 (159% degradation, the most severe of the three ablations) / TV-A 25.08 / FAD 4.20 (53% degradation). The failure manifests as severe background degradation — consistent with expectations, since SC is the core mechanism for anchoring source context and preserving non-target regions.
  · W/o SIGA (removing source-instruction gated attention): FVD 187.28 (only a mild 3.8% degradation) / TV-A 24.90 / FAD 3.26 (18.5% degradation). The failure manifests as audio hallucinations and stuttering. This shows SIGA's effect is mainly on the audio side and instruction-following quality, with little impact on visual fidelity.
  · W/o Two-Stage Training Strategy (removing the two-stage curriculum): FVD 291.55 (61.6% degradation) / TV-A 24.87 / FAD 5.18 (88.4% degradation, the largest audio degradation of the three). The failure manifests as visual distortion and inconsistency.
  The common feature of the three ablations is that TV-A (text-video alignment) changes very little (25.23 → 25.08/24.90/24.87), indicating that basic instruction-following ability mainly comes from the base model and the data itself, while architectural changes mainly affect fidelity and consistency.

[Completely missing data-side ablations] [Uncertain]
  · Filtering-strictness ablation: no sensitivity analysis is done for the CoTracker3 motion threshold, LAION aesthetic threshold, Audiobox threshold, or the MLLM five-dimension threshold; no "loose vs. strict filtering" performance comparison.
  · Data-scale ablation: no 25%/50%/100% data-scale scaling experiments are done, so the question "has 79K already saturated, would further expanding the synthesis scale still yield gains" cannot be answered — the most important question a work touting "scalable synthesis" should answer.
  · Automatic-verification-step ablation: no comparison of training effects "with/without Qwen3-Omni five-dimension verification," so the actual value of this verification step cannot be quantified, nor can it be verified whether the strict conjunctive condition of "must pass all five simultaneously" is too strict (possibly wrongly eliminating many usable samples).
  · Data-engine component ablation: no comparison of the effect on the downstream model of data pairs synthesized "with/without frame-wise cross-attention audio conditioning," so the benefit of enforcing synchronization at the synthesis stage cannot be quantified.
  · Bidirectional-instruction ablation: whether the reverse samples brought by instruction_reverse actually help mitigate synthetic distribution bias is not verified (the paper's main text does not even mention this design).
  · Five-subset-mixture ablation: no comparison of the effect of different task mixture ratios, nor any per-task breakdown of performance reported.
  · Caption/instruction-style ablation: no comparison of different instruction-generation strategies (e.g., templated vs. Qwen3-Omni free generation).
  Overall assessment: this work's model-architecture ablations are thorough and clear-cut, but almost no experimental verification is done for its data contribution; the pipeline's effectiveness can only be indirectly supported by "the final model performs well," with no way to decompose the independent contribution of each data design.

### Evidence for Quality vs. Quantity (Cases Where Small-but-Curated Data Outperforms Large-but-Noisy Data) ⚠️

[Uncertain] The paper performs no experiments weighing data scale against quality, and provides no direct evidence.

[Indirect design choices reflecting a "quality over quantity" orientation]
  · Strict conjunctive acceptance: automatic verification requires samples to pass all five dimensions simultaneously (instruction fidelity / content preservation / perceptual quality / AV sync / safety), with elimination on failing any single one. This is a clear "better to have fewer but good" orientation, far more conservative than a weighted-sum soft filter.
  · Stringent material admission: speech samples must satisfy "speech temporally aligned with a visible on-screen speaker," and non-speech samples must satisfy "a single clear dominant sound source" — both highly exclusive hard conditions that would eliminate a great deal of material that could otherwise be marginally usable.
  · Triple manual gatekeeping of the evaluation set: 1K samples reviewed by 20 people, 10 judge pairs, independent per-dimension review, collected only if all five criteria pass.
  · Restraint in the data scale itself: 79K pairs (about 110 hours) is a small dataset for fine-tuning a 5B-class model, far smaller than the million-scale typical of peer works. The model's ability to outperform baselines at this scale indirectly suggests the value of high-quality controlled pairing — but without a control group, it cannot be ruled out that the contribution comes from architectural improvements (SC/SIGA) rather than data quality.

[An opposing tension that exists] On the other hand, the paper touts "scalable" as the pipeline's core selling point, implying its value proposition partly rests on "being able to make more data" — which is somewhat in tension with a "small but curated" narrative. The paper does not clarify whether 79K is a deliberately controlled scale ceiling (quality-first) or the compute ceiling at the time (still wanting to make more).

[The experiment most worth doing but not done] Using the same pipeline at different verification strictness levels to produce datasets of different sizes (e.g., "pass only 3 criteria" yields 200K pairs vs. "pass all five" yields 79K pairs), and comparing training effects — this experiment would both demonstrate the value of strict acceptance and answer the question of "where is the scale-quality optimum for synthetic data," a question of broad significance for the synthetic-data paradigm in general.

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g., VABench's Seven Major Categories) ⚠️

There is partial alignment between the organizing dimension of the training data and the evaluation system, but the alignment axis is "editing failure mode" rather than visual content category, and fine-grained per-task alignment is missing.

[Alignment relationship one: data-acceptance dimensions ↔ evaluation metrics (well aligned)]
  The five automatic-verification dimensions correspond clearly to the 11 evaluation metrics:
  · instruction fidelity ↔ TV-A (text-video alignment, VideoCLIP-XL), TA-A (text-audio alignment, CLAP)
  · content preservation ↔ SSIM (structural similarity), TC (temporal consistency, CLIP inter-frame similarity), LPAPS (audio perceptual similarity)
  · perceptual quality ↔ FVD (Fréchet Video Distance), FAD (Fréchet Audio Distance, VGGish)
  · audio-video synchronization ↔ AV-A (ImageBind), PEAVS, Sync-C, Sync-D (SyncNet, vshift=15)
  · safety ↔ no corresponding evaluation metric (safety is not measured at the evaluation stage, the only unaligned dimension)
  This "filter data the same way the model is evaluated" consistency design is one of the more self-consistent aspects of this work's methodology; the 11 metrics are grouped into video (4)/audio (3)/audio-video (4) categories, with complete coverage.

[Alignment relationship two: training-task types ↔ evaluation-task types (insufficiently aligned)]
  Training data is split into five subsets (add_and_remove / clone_id / clone_voice / clone_id_voice / general_editing), and the main paper text summarizes them into four scenario types (identity-preserving speech modification / AV instance editing / instance insertion / instance removal), but evaluation reports only overall metrics on the mixed evaluation set, without breaking results down by task type. This means it cannot be seen which type of editing the model is strong or weak at — for example, "instance insertion" (needing to generate a new object and its sound out of nothing) is clearly of different difficulty than "instance removal" (needing to repair the background); the mixed metric masks this difference. This is a clear shortcoming of the evaluation design.

[External benchmark alignment] Beyond the self-built 1K evaluation set, additional zero-shot generalization evaluation is performed on the public AvED-Bench (FVD 227.82 / FAD 4.32 / AV-A 23.71, compared to AVI-Edit's 372.37 / 7.65 / 23.21). This step is important: since the training set and the self-built evaluation set come from the same data engine, a systematic bias of "evaluation favoring one's own synthetic style" exists; the leading results on the external benchmark provide independent corroboration of the model's true generalization capability.

[Relationship to visual domain taxonomy systems] This work does not involve a VBench/VABench-style visual-content taxonomy (people/actions/scenes/styles, etc.) — the training data is neither organized nor evaluated along such lines — because the capability boundary of the editing task is defined by "editing operation type" rather than "content category." This is the fundamental divide between editing-type and generation-type works in terms of evaluation systems.
[Uncertain] No alignment analysis is done with mainstream video-generation benchmarks such as VBench or VABench; no per-task-type breakdown metrics are reported; there is no explicit statement that the 1K evaluation set and the 79K training set have no overlap at the source-material level.

## Uncertain Fields

The research information for the following fields is partially uncertain (marked with ⚠️ in the source):

- release_date
- openness
- data_scale
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
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
