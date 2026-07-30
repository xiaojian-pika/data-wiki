# The Open-Sora series (Open-Sora 1.0/1.1/1.2/1.3/2.0, HPC-AI Tech) and the Open-Sora Plan series (v1.0-v1.5, Peking University PKU-YuanGroup). These are two independent projects, often referred to together as the "two champions of open-source Sora reproduction": Open-Sora is led by HPC-AI Tech (Luchen Technology), focused on extremely low-cost training (the 2.0 version cost $200k); Open-Sora Plan is led by Peking University's Yuan Li research group, focused on community-collaborative reproduction and a multi-dimensional data-cleaning funnel. This entry combines research on both; wherever field content differs between the two, it is itemized separately.

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

The Open-Sora series (Open-Sora 1.0/1.1/1.2/1.3/2.0, HPC-AI Tech) and the Open-Sora Plan series (v1.0-v1.5, Peking University PKU-YuanGroup). These are two independent projects, often referred to together as the "two champions of open-source Sora reproduction": Open-Sora is led by HPC-AI Tech (Luchen Technology), focused on extremely low-cost training (the 2.0 version cost $200k); Open-Sora Plan is led by Peking University's Yuan Li research group, focused on community-collaborative reproduction and a multi-dimensional data-cleaning funnel. This entry combines research on both; wherever field content differs between the two, it is itemized separately.

### Releasing institution/company

(1) Open-Sora: HPC-AI Tech / Luchen Technology (the Colossal-AI team), GitHub organization hpcaitech. (2) Open-Sora Plan: Peking University's Yuan Li research group (PKU-YuanGroup), in collaboration with Peng Cheng Laboratory, Rabbitpre AI (兔展智能), and others; its HuggingFace organization is LanguageBind. The two projects have no affiliation with each other and merely share a similar name.

### Release date (technical report/paper/open-source date)

[Open-Sora (HPC-AI Tech)] v1.0: March 2024; v1.1: April 25, 2024 (first release of the complete video data-processing pipeline); v1.2: June 17, 2024 (corresponding to paper arXiv:2412.20404 "Open-Sora: Democratizing Efficient Video Production for All," posted to arXiv on December 29, 2024); v1.3 (1B): February 20, 2025; v2.0 (11B): released March 12, 2025, technical report arXiv:2503.09642 "Open-Sora 2.0: Training a Commercial-Level Video Generation Model in $200k."
[Open-Sora Plan (Peking University)] v1.0.0: April 2024; v1.1.0: May 2024; v1.2.0: July 2024; v1.3.0: October 2024; technical report arXiv:2412.00131 "Open-Sora Plan: Open-Source Large Video Generation Model" submitted November 28, 2024 (content corresponds to v1.3); v1.5.0: released June 5, 2025 (8B, SUV sparse MMDiT + 8×8×8 WFVAE).

### Type (model/dataset/toolchain/evaluation benchmark)

Both are open-source projects with the dual nature of "model + complete toolchain": both release model weights (T2V/I2V diffusion models + VAE) and end-to-end training code and data-processing code, along with some annotated datasets. Neither is an evaluation benchmark. Open-Sora additionally has the property of a "training-cost engineering template" (stage-by-stage cost accounting); Open-Sora Plan additionally has the property of a "community-collaborative reproduction template."

### Degree of openness (whether weights/code/data/pipeline are each open source) ⚠️

Both belong to the highest tier of openness in the video-generation field, but neither achieves "fully open-sourced datasets."
[Open-Sora (HPC-AI Tech)] Weights: open source (HuggingFace hpcai-tech, Apache 2.0); code: open source (inference + training + distributed optimization); data-processing pipeline: fully open source, which is this project's most valuable reference point — under the v1.1/v1.2 branches, the tools/ directory contains scene_cut (PySceneDetect shot detection + splitting), scoring (four scorers: aesthetic / optical_flow / ocr / matching), caption (PLLaVA, LLaVA, LLaMA3 captioning), and datasets (datautil filtering and cleaning), along with docs/data_processing.md that ties the whole pipeline together, with every step accompanied by directly runnable torchrun commands and threshold examples; the training data itself: not open source (only the names of the source datasets and the filtering thresholds are disclosed; the filtered meta files are not released). Note: the main branch of version 2.0 has removed the tools/ data-processing directory (tools/datasets, tools/scoring, and docs/data_processing.md all return 404 on the main branch); the complete data-processing code must be obtained from historical tags such as v1.2.0; the new pipeline described in the 2.0 technical report (PaddleOCR, VMAF, Laplacian, etc.) has no corresponding open-source implementation on the main branch — the degree of openness here is actually a **regression**.
[Open-Sora Plan (Peking University)] Weights: open source; code: open source (training + inference + WFVAE); data: partially open source — each version has released "Data and Annotations" annotation data and prompt_refiner datasets on HuggingFace (LanguageBind/Open-Sora-Plan-v1.1.0 / v1.2.0 / v1.3.0); data-processing pipeline: the paper and Report documents give complete filtering steps, tools, thresholds, and stage-by-stage retention rates (a rare quantitative disclosure across the whole industry), but no independently packaged curation code repository is available — reproduction requires manually assembling it from the documentation. [Uncertain]

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Neither supports it. The entire Open-Sora and Open-Sora Plan series (as of Open-Sora 2.0 / Open-Sora Plan v1.5.0) are purely visual (silent) video-generation models whose outputs have no audio track; none of the technical reports, GitHub documentation, or data pipelines contain any design or description of an audio encoder, audio latent, joint audio-video denoising, or audio-video alignment. The data-processing side likewise does not touch the audio track at all — splitting, scoring, and captioning are all done purely on visual frames. Accordingly, all audio- and audio-video-alignment-related fields in this entry (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling) are "Not applicable."

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

[Official first-hand] 1) Open-Sora 2.0 technical report arXiv:2503.09642 https://arxiv.org/abs/2503.09642 and the full HTML text https://arxiv.org/html/2503.09642v1 (data pipeline, data statistics charts, three-stage cost table); 2) Open-Sora 1.2 paper arXiv:2412.20404 https://ar5iv.labs.arxiv.org/html/2412.20404 (data sources, 80k-hour statistics, bucket strategy, 35k H100 GPU-hours); 3) Open-Sora main GitHub repository https://github.com/hpcaitech/Open-Sora (version timeline, scope of open-sourcing); 4) Open-Sora v1.2.0 data-processing code and documentation (first-hand code-level evidence): docs/data_processing.md, tools/scoring/README.md, tools/scene_cut/README.md, tools/caption/README.md, with raw paths such as https://raw.githubusercontent.com/hpcaitech/Open-Sora/v1.2.0/tools/scoring/README.md; 5) Open-Sora Plan paper arXiv:2412.00131 https://arxiv.org/html/2412.00131v1 (data-source table, seven-level filtering funnel and stage-by-stage retention-rate table, training-stage table); 6) Open-Sora Plan GitHub https://github.com/PKU-YuanGroup/Open-Sora-Plan and docs/Report-v1.3.0.md, docs/Report-v1.5.0.md (thresholds, 27% retention rate, prompt-refiner details, v1.5 data scale).
[Third-party reporting] 7) MarkTechPost's report on the Open-Sora 2.0 release https://www.marktechpost.com/2025/03/14/hpc-ai-tech-releases-open-sora-2-0-an-open-source-sota-level-video-generation-model-trained-for-just-200k/; 8) HuggingFace Papers pages https://huggingface.co/papers/2503.09642 and https://huggingface.co/papers/2412.00131; 9) comfyui-wiki release bulletin https://comfyui-wiki.com/en/news/2025-03-13-open-sora-2-release.

## Data Scale and Distribution

### Training data magnitude (video count/hour count/token count, pretraining and SFT listed separately) ⚠️

[Open-Sora 2.0] Data volume is given by training stage (the paper's table reports "number of videos participating in that stage's training"): stage 1, 70M videos (256px T2V); stage 2, 10M videos (256px T2V/I2V); stage 3, 5M videos (768px T2V/I2V). The report does not give the total volume of raw video before cleaning, total hours, or token count, nor does it list an SFT scale separately (the stage-3 5M high-definition subset is effectively the high-quality fine-tuning set).
[Open-Sora 1.2] The original pool is about 30M video clips (2-16 seconds long), totaling about 80k hours; of these, the Panda-70M high-quality subset is 20M clips, about 41k hours; the final high-quality stage is about 2M clips, 5k hours. Images: about 3M.
[Open-Sora Plan v1.3] Images: 18.0M; videos: about 28M (before cleaning); after cleaning, the Panda70M portion retains about 19M clips (a 27% retention-rate figure).
[Open-Sora Plan v1.5] Images: 1.1B (only a resolution check is performed, no quality filtering); videos: 40M high-quality samples.
Note: all of the above are item-count figures; neither project has published training token counts, and Open-Sora 2.0 has not published total hours. [Uncertain]

### Data source composition (proprietary/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

Both projects **rely entirely on public datasets and free, freely-licensed stock-footage sites, with no proprietary copyrighted library, no purchased/licensed data, and no synthetic data** — this is the most essential difference from closed-source models such as Sora 2 / Veo 3 / Seedance, and also the reason their data recipes are fully reproducible.
[Open-Sora 1.2 explicitly lists] Webvid-10M (10 million video-text pairs, stock footage), Panda-70M (70 million pairs, from which a 20M high-quality subset is taken), HD-VG-130M (130 million pairs, BLIP-2 captions), MiraData (77,000 long videos, gaming/city walkthroughs), Vript (400,000 densely annotated videos), Inter4K (1,000 4K clips), plus free-license stock sites such as Pexels / Pixabay / Mixkit; images use a LAION subset (aesthetic score > 6.5) and Unsplash-lite.
[Open-Sora 2.0] The technical report **does not disclose** data sources or specific dataset names, only describing the filtering method — a notable information gap in the report (presumably continuing the 1.x public-dataset mix, but without official confirmation). [Uncertain]
[Open-Sora Plan v1.3] Images: SAM 11.1M (paired with LLaVA captions), the English subset of Anytext-3M at 1.8M (about 50% of that dataset), 160,000 high-quality portrait images filtered from LAION-5B, and 5.0M internally Qwen2-VL-annotated images; videos: Panda70M 21M (landscape), VIDAL 3M (portrait, from YouTube Shorts), ShareGPT4Video 0.8M (CC0 footage sourced from Mixkit / Pexels / Pixabay).
[Open-Sora Plan v1.5] Images come from Recap-DataComp-1B, COYO-700M, and LAION-Aesthetics; videos come from Panda-70M and "internal sources" — v1.5 introduces undisclosed internal video data for the first time, a regression in data transparency for this series.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Both projects handle training-data compliance and provenance weakly — a typical approach for academic/open-source projects:
- They rely on the licenses of the third-party public datasets themselves (Panda-70M is sourced from YouTube videos in HD-VILA-100M, Webvid-10M is sourced from watermarked Shutterstock footage, VIDAL is sourced from YouTube Shorts); the copyright status of these datasets is itself disputed, and the project teams performed no secondary licensing verification and did not disclose the proportion of licensed data.
- The portion of ShareGPT4Video used by Open-Sora Plan is explicitly labeled as CC0 copyright-free footage sourced from Mixkit / Pexels / Pixabay, and Open-Sora 1.2 likewise explicitly uses free-license footage from Pexels / Pixabay / Mixkit — this is the only portion of either project that could be called rights-cleared, but neither gives a proportion figure.
- Neither uses any purchased/licensed data, and neither has built a rights-cleared dataset inventory.
- On the output side: neither implements C2PA metadata, invisible watermarking, or generated-content detection tools; model weights are released under permissive licenses such as Apache 2.0, with no provenance constraints on downstream use.
- Notably, the Webvid-10M footage carries Shutterstock watermarks, and Open-Sora 1.2 uses it for stage-1 low-resolution pretraining (240p/360p), meaning watermark signal enters the model's early training — a well-known common problem in open-source reproductions, whose impact the project documentation does not discuss. [Uncertain]

### Clip duration distribution and splitting strategy

[Open-Sora 2.0] The preprocessing stage first removes raw videos shorter than 2 seconds; after shot splitting, clips longer than 8 seconds are forcibly cut into multiple 8-second segments, and clips shorter than 2 seconds are discarded, so training-clip duration falls strictly within [2s, 8s]. The paper's Figure 3 statistics show the duration distribution is markedly right-skewed, with **nearly half of clips concentrated in the 6-8 second range**. Output clips are also constrained to fps < 30.
[Open-Sora 1.x] Clip duration is 2-16 seconds, combined with a bucket mechanism that buckets by frame count, supporting variable-length training from 2s to 15s.
[Open-Sora Plan v1.3] Uses ffmpeg to uniformly cut into 16-second clips, then uses LPIPS for jump-cut detection, keeping only clips whose frame count falls within [32, 512] (about 1.3-21 seconds at 24fps).
[Open-Sora Plan v1.5] Video training progressively increases from 57 frames @24fps (about 2.4 seconds) up to 121 frames @24fps (about 5 seconds).

### Resolution/aspect-ratio distribution and bucketing strategy

[Open-Sora 2.0] Preprocessing removes videos whose aspect ratio falls outside [1/3, 3], and constrains clips' long edge to ≤1080px, uniformly transcoded to H.264. Figure 3 shows aspect ratio (height/width) is **mostly concentrated in the 0.5-0.75 range, i.e. dominated by 16:9 landscape**. Inference supports two tiers, 256px and 768px, covering 16:9, 9:16, 1:1, and 2.39:1.
[Open-Sora 1.x] Uses an explicit **bucketing strategy**: resolution, aspect ratio, and frame count are combined into a three-dimensional predefined bucket, and each bucket is given its own batch size to balance GPU load, allowing a single training run to mix samples from 144p to 720p, any aspect ratio, and 2-15 seconds. This is one of Open-Sora's most widely reused engineering designs.
[Open-Sora Plan v1.3] Video training is fixed at 93×352×640 (about 16:9); v1.5 performs multi-resolution training on the image side, covering five aspect ratios — (1,1), (3,4), (4,3), (9,16), (16,9) — combined with a Min-Max Token strategy, while the video side is fixed at a 9:16 ratio, up to 121×576×1024.
[Ratio control on the data-source side] Open-Sora Plan explicitly balances landscape vs. portrait: Panda70M's 21M provides landscape, VIDAL's 3M provides portrait (YouTube Shorts) — a rare explicit strategy for supplementing portrait-orientation data.

### Category/domain distribution and mixture strategy (proportion control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

Neither project does **explicit category/domain mixture control or concept balancing** — this is the most obvious shortcoming of their data strategy relative to industry models (e.g. Seedance, Movie Gen, which have fine-grained domain taxonomies and mixture tables).
Identifiable implicit domain structure:
- Open-Sora 1.2 achieves domain diversity indirectly by mixing data from different sources: Webvid/Panda-70M provides general web video (the YouTube long tail), MiraData specifically supplements gaming footage and long city-walkthrough shots, Vript supplements densely annotated cinematic content, Inter4K supplements 4K high-definition material, and LAION/Unsplash supplements static high-aesthetic images. This is a "division of labor by dataset function" approach rather than a "mixture by semantic category" approach.
- Open-Sora Plan specifically filters 160,000 high-quality portrait images out of LAION-5B to strengthen character-generation ability — the only place with an explicit, category-targeted supplement; VIDAL's 3M portrait Shorts implicitly carry a large amount of person-talking/lifestyle content.
- Neither publishes proportion figures for people/actions/scenes/styles, neither does long-tail concept balancing, and neither does resampling after semantic clustering. The Open-Sora 2.0 report only provides statistical charts (Figure 3) along four low-level dimensions — aesthetic score/duration/aspect ratio/caption length — with no statistics along any semantic-category dimension.
- The style dimension exists only as a descriptive field within captions (one of Open-Sora 2.0's six elements, "video style"), but is not used as a basis for training-set mixture.
Conclusion: domain distribution is basically a passive inheritance of the source datasets' distribution rather than an active design choice. [Uncertain]

### Audio category distribution and mixture (proportion and control strategy for speech/sound effects/music/ambient sound/silence) — a dimension unique to AV models

Not applicable. The entire Open-Sora and Open-Sora Plan series do not generate audio; the training-data pipeline does not process audio tracks, and there is no classification, mixture, or statistics for speech/sound effects/music/ambient sound.

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track)

Both projects' data consists of **strictly single-shot clips**, a core design orientation of their pipelines:
- Open-Sora 2.0 uses FFmpeg libavfilter's scene score for shot-boundary detection and splitting, then layers on PySceneDetect-based camera-shake/cut detection as a secondary removal step, ensuring no shot changes within a clip; long clips are mechanically cut into 8-second segments.
- Open-Sora 1.x uses PySceneDetect to detect and split scenes, with output named `{video_id}_scene-{scene_id}.mp4`, likewise single-scene clips.
- After the 16-second splitting step, Open-Sora Plan v1.3 specifically adds a level of **LPIPS jump-cut detection**, using sudden changes in perceptual distance to identify cut points, with a 97% retention rate; the explicit goal is to filter out clips containing shot changes.
Therefore: multi-shot narrative samples are actively removed rather than retained; the average shot count in the training data is 1; there is no cross-shot consistency supervision signal, and the models do not have multi-shot narrative-generation capability. Average clip duration: Open-Sora 2.0 skews toward 6-8 seconds, Open-Sora Plan is about 1.3-21 seconds (retained by frame-count range after 16-second splitting). Presence of native audio track: the source data does have audio tracks, but these are completely ignored by the pipeline and dropped during transcoding.

### Language/accent distribution (data foundation for multilingual lip-sync capability)

Not applicable at the speech level (no audio generation). On the text side: Open-Sora Plan explicitly uses only the **English subset** of Anytext-3M (about 50% of that dataset, 1.8M), and captions and prompts are entirely in English; captions across the Open-Sora series are likewise entirely in English (PLLaVA, LLaVA-Video, and Qwen2.5-Max all output English descriptions). Neither project has built data for multilingual prompt support; Chinese prompts require external translation. Accent/lip-sync-related data foundation: none.

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of levels, ordering)

Both projects' cleaning funnels are clearly structured and fully open-sourced/documented, making them the most reproducibility-valuable samples on this topic.
[Open-Sora 2.0 (hierarchical data pyramid)] The design philosophy is that "filtering thresholds tighten progressively from loose to strict, forming a pyramid-shaped data hierarchy that corresponds one-to-one with a progressive training curriculum that goes from low resolution to high resolution" — the base layer is a large volume of loosely filtered data for low-resolution pretraining, while the top layer is a small volume of strictly filtered data for high-resolution fine-tuning. Specific levels:
  Level 0, preprocessing: remove corrupted files and outlier samples (duration < 2s, bpp < 0.02, fps < 16, aspect ratio outside [1/3, 3]);
  Level 1, shot splitting: FFmpeg libavfilter's scene score detects shot boundaries and splits, output uniformly at fps < 30 / long edge ≤ 1080px / H.264, clips > 8s are cut into 8s segments, clips < 2s are discarded;
  Level 2, aesthetic score: CLIP+MLP aesthetic scorer, averaging the first/middle/last frames;
  Level 3, motion score: FFmpeg libavfilter's VMAF motion score, removing both extremes (too low = static, too high = violent/chaotic);
  Level 4, blur detection: OpenCV Laplacian-operator variance threshold, with a majority vote over five uniformly sampled frames per video;
  Level 5, OCR: PaddleOCR detects text with confidence > 0.7, and clips with too much text are discarded;
  Level 6, camera shake: PySceneDetect's Shot Boundary Detection — frame-to-frame average change exceeding a threshold is judged as shake and removed;
  Level 7, captioning: LLaVA-Video for the low-resolution stage, Qwen2.5-Max for the high-resolution stage.
[Open-Sora 1.x (open-source code-level pipeline, docs/data_processing.md)] Four stages: (1) shot splitting (PySceneDetect scene_detect → cut); (2) quality assessment and filtering (aesthetic → optical flow → OCR → datautil filtering); (3) captioning and alignment (PLLaVA/LLaVA captioning → CLIP matching score to compute image-text consistency → caption cleaning); (4) camera-motion detection performed on the remaining samples, with results written back into the caption.
[Open-Sora Plan v1.3 (seven-level funnel, stage-by-stage retention rates fully disclosed)] ffmpeg 16s splitting → LPIPS jump-cut detection → LPIPS motion filtering → EasyOCR subtitle cropping → LAION aesthetic filtering → DOVER technical-quality filtering → LPIPS motion re-check. A special feature is that **motion filtering is performed twice**: a coarse pass first, then a re-check after the subtitle region has been cropped out, because cropping changes the frame composition and may turn a clip that originally "had motion" into an effectively static one.

### Quantitative funnel retention rate (input/output volume at each level and final retention rate, e.g. Apollo's 27%) ⚠️

[Open-Sora Plan v1.3 — a rare stage-by-stage quantitative disclosure across the whole industry] The paper gives a complete seven-level retention-rate table (cumulative retention relative to the raw data):
  Video splitting (ffmpeg, 16s) → 100%
  Jump-cut detection (LPIPS, keeping 32 ≤ frame count ≤ 512) → 97%
  Motion filtering (LPIPS, 0.001 ≤ score ≤ 0.3) → 89%
  OCR subtitle cropping (EasyOCR, edge ratio 0.20) → 89% (this level only crops and does not discard, so the retention rate is unchanged)
  Aesthetic filtering (LAION Aesthetic v2, ≥ 4.75) → 49% [the single most aggressive level, cutting about 40 percentage points]
  Technical quality (DOVER technical score ≥ 0) → 44%
  Motion re-check (LPIPS, 0.001-0.3) → 42%
  **Final cumulative retention rate is about 42%**.
According to another figure from the Open-Sora Plan v1.3.0 Report, the cleaned Panda70M **retained only 27% of the original data** (about 19 million clips out of an original approximately 70 million), which is highly consistent in magnitude with the 27% reported by Apollo. The two figures use different bases (42% is the cumulative figure from the paper's funnel table, 27% is the final figure specifically for the full Panda70M set), so the source version should be noted when citing either.
[Open-Sora 2.0] Explicitly states that a "hierarchical data pyramid" was built to match the progressive training curriculum, but **gives no input/output volume or retention-rate figures for any filtering level**; the pyramid's tier ratio can only be inferred indirectly from the per-stage training data volumes (70M → 10M → 5M) at roughly 14 : 2 : 1, i.e. the highest-quality tier is only about 7% of the base tier.
[Open-Sora 1.2] Does not give stage-by-stage retention rates; a rough ratio can be inferred as the original 30M clips (80k hours) → the final high-quality stage's 2M clips (5k hours), about 6.7%. [Uncertain]

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting)

[Open-Sora 1.x] Uses **PySceneDetect**, with fully open-source code: `python tools/scene_cut/scene_detect.py meta.csv` outputs a list of scene timestamps for each video (of the form `[('00:00:01.234','00:00:02.345'), ...]`), which `tools/scene_cut/cut.py` then uses to split by timestamp, producing output named `{video_id}_scene-{scene_id}.mp4`; before this, `convert_id_to_path.py` performs corrupted-file filtering (outputting an intact-flag column).
[Open-Sora 2.0] Switches to **FFmpeg libavfilter's scene score** for shot-boundary detection (faster than PySceneDetect, and can be done within the same pipeline as transcoding), then still uses **PySceneDetect's Shot Boundary Detection** as a secondary detector for camera shake/abnormal cuts. A fixed 8-second truncation is applied after splitting.
[Open-Sora Plan] Does not perform content-based shot splitting; instead it first uses **ffmpeg to mechanically cut into fixed-length 16-second clips**, then uses **LPIPS perceptual distance** to detect whether a jump cut exists within a clip, discarding the entire clip if it contains a cut (retention rate 97%). This is a different technical route from the former: "detect and discard" in place of "detect and split at the cut point" — simpler to implement, but it loses some usable footage.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection)

[Aesthetics] Both use LAION-family CLIP+MLP aesthetic scorers (the improved-aesthetic-predictor, trained on 176K SAC + 15K LAION-Logos + 250K AVA image-text pairs, outputting a 1-10 score). The empirical scale given in the Open-Sora documentation: 5.5 is the "acceptable" threshold, 6.5 is the "high-aesthetic" threshold, and a good text-to-image model can reach 7.0+; videos are evaluated on the first/middle/last frames; the example filtering command is `python -m tools.datasets.datautil meta.csv --aesmin 5.0`. Open-Sora 1.2's filtering threshold for Panda-70M is **aesthetic score > 4.5** (yielding a 20M subset / 41k hours); on the image side, the LAION subset uses **aesthetic score > 6.5**. Open-Sora Plan uses **LAION Aesthetic v2 ≥ 4.75** (the paper explicitly explains that 4.75 was chosen because it also happens to filter out a large number of frames with excessive text), and for **samples with aesthetic score > 6.25, a "A high-aesthetic scene" prefix is automatically added before the caption** — turning the aesthetic score into a controllable condition rather than only a filter. Open-Sora 2.0's aesthetic-score distribution statistics show the training-data bulk falls at **4.5-5.5** (i.e. a large number of "medium-aesthetic" samples were deliberately retained to preserve diversity, rather than keeping only high-scoring samples).
[Sharpness/blur] Open-Sora 2.0 uses the **variance of the OpenCV Laplacian operator** for blur detection, judging five uniformly spaced frames per video and taking a majority vote; Open-Sora Plan uses **DOVER's technical-branch predicted score (threshold ≥ 0)**, specifically targeting compression artifacts and low-bitrate sources, closer to real quality degradation than a pure blur operator. Open-Sora 2.0 additionally uses **bpp (bits per pixel) < 0.02** during preprocessing to directly remove low-bitrate sources.
[OCR / text filtering] Three different routes: Open-Sora 1.x uses **DBNet++ (the MMOCR implementation)**, with an output column `ocr` recording **the number of text regions detected with confidence > 0.3**, used to remove densely texted scenes such as news broadcasts and advertisements; Open-Sora 2.0 uses **PaddleOCR**, counting only text with confidence > 0.7, discarding clips with too much text; Open-Sora Plan uses **EasyOCR, detecting only the bottom 20% of the frame** (the typical location of subtitles), and adopts a **crop-rather-than-discard** strategy (edge ratio 0.20); the paper also candidly admits this method's limitation — it cannot handle advertisement text or presentation subtitles located in the center of the frame.
[Black bars/watermarks/logos] None of the three implements a dedicated black-bar-detection, watermark-detection, or logo-detection module — a shared gap; the Shutterstock watermark issue in Webvid-10M therefore goes unaddressed.

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content)

[Open-Sora 1.x] Uses the **UniMatch (GMFlow) optical-flow model** to compute a dense optical-flow score (`tools/scoring/optical_flow/inference.py`, output column `flow`) — the higher the score, the greater the motion — used to remove static/near-static clips; the optical-flow result is also used for **camera-motion detection** (identifying pan left / zoom in, etc.), and high-confidence camera-motion labels are appended into the caption. The camera-motion-detection module is separately open-sourced under tools.
[Open-Sora 2.0] Switches to **FFmpeg libavfilter's VMAF motion score** (much faster than an optical-flow model, and can be computed incidentally within the transcoding pipeline), with **bidirectional removal**: clips with extremely low motion scores (static footage) and extremely high ones (violent shaking/chaotic cuts) are both discarded. The motion score is also **appended into the caption text**, allowing the motion magnitude of the generated video to be controlled via the prompt at inference time.
[Open-Sora Plan] Uses **LPIPS inter-frame perceptual distance** in place of optical flow as the motion metric (far cheaper to compute than optical flow), with a retention interval of **0.001 ≤ score ≤ 0.3**: below 0.001 is considered near-static, and above 0.3 means "clear jitter and flicker are present." The authors state this threshold was verified as acceptably accurate via **manual spot-checking of 2,000 videos**. And as noted earlier, after subtitle cropping a **second motion re-check** is run (retention drops from 44% to 42%).
Common to all three: motion filtering is bidirectional (removing both static and jittery content), and the motion score is ultimately used as a controllable condition rather than only as a filter.

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Neither project **implements a substantive deduplication step** — a clear gap in their data pipelines relative to industry practice.
- Open-Sora's tools/ directory has no independent deduplication module; although the README's feature list mentions deduplication, the four-stage flow in v1.2.0's docs/data_processing.md contains no deduplication step, and tools/datasets/datautil mainly performs score-column-based filtering and meta merging, with no embedding-level semantic deduplication implementation to be found.
- Open-Sora Plan's seven-level funnel table likewise contains no deduplication level, and neither the paper nor any version's Report discusses exact deduplication (hashing) or semantic deduplication (embedding clustering).
- Potential risk: Panda-70M's 3.8 million long videos from HD-VILA-100M are cut into 70.8 million clips, and adjacent clips from the same long video are highly similar; Open-Sora Plan additionally layers in ShareGPT4Video (also sourced from Pexels/Pixabay), and neither source overlap nor intra-clip redundancy is addressed. [Uncertain]

### VLM/LLM as quality inspector (multimodal-large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

Both remain **stuck at the "shallow scorer" stage and have not yet shifted to the "large-model semantic judgment" paradigm** — every filtering criterion (aesthetic score, optical-flow/VMAF/LPIPS motion score, Laplacian blur, DOVER technical score, OCR text count) is a scalar score plus threshold output by a lightweight, task-specific model or a traditional CV operator; no filtering level is performed by VLM/LLM semantic-level judgment.
The only mechanism that comes close to "model as judge" is **Open-Sora 1.x's matching score (image-text matching score)**: it uses **CLIP** to compute the cosine similarity between the video's middle frame and its caption (`tools/scoring/matching/inference.py`, output column `match`), used to remove image-text-mismatched samples. But CLIP's semantic discriminative power is far weaker than that of contemporary VLM judges, and it only takes a single frame, ignoring temporal information.
The VLM's role in both projects is that of a **captioner rather than a judge** (PLLaVA-13B / LLaVA-Video / Qwen2-VL-7B / Qwen2.5-Max are responsible only for generating captions; their outputs are not used to score sample quality or make removal decisions). Open-Sora Plan's Qwen2-VL caption post-processing likewise only uses a rule table of 28 common opening phrases to strip prefixes like "The video shows" — a rule-based cleanup rather than model judgment.
Significance of the comparison: this is precisely where the main generational gap lies between 2024-2025 open-source reproduction projects and 2026 frontier practice — the Open-Sora series' pipeline can be viewed as a complete technical snapshot of the "pre-VLM-as-judge" era.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

Neither project's technical report or documentation **describes any NSFW detection, violent-content filtering, face/privacy protection, or copyrighted-content detection step**; safety filtering is effectively entirely absent, implicitly relying on the safety cleaning already performed by the upstream public datasets themselves (Panda-70M, LAION, Webvid, etc.). The only indirectly related items are: Open-Sora Plan applies a "high-quality" filter when selecting portrait images from LAION-5B, but without any privacy/portrait-rights consideration; both projects' OCR filtering removes densely texted scenes, which incidentally filters out some material bearing copyright notices/station logos, but this is a side effect rather than a design goal. Model weights are released under permissive licenses such as Apache 2.0, with no accompanying output-side safety classifier or watermark. This is the biggest compliance gap between open-source reproduction projects and closed-source commercial models (e.g. Sora 2's CSAM source-level screening plus multi-tier safety classifiers). [Uncertain]

## Captioning Approach

### Caption models used (in-house VLM/open-source model, model scale)

[Open-Sora 1.x] The primary model is **PLLaVA 13B** (the 13B version is chosen to balance speed and quality, configured with 2×2 spatial pooling, pooling_shape 4-12-12, with input uniformly sampling 4 frames from the video). The documentation explicitly explains why GPT-4V is not used: "GPT-4V works better, but 20 seconds/sample is too slow for us" — a typical record of the captioning-cost trade-off in open-source projects. It also provides an LLaVA v1.6-Mistral-7B captioning script and uses **LLaMA 3** to extract objects/tags from captions, with code found in tools/caption/. Earlier versions also mention GPT-4V as an optional high-quality path.
[Open-Sora 2.0] Uses different models tiered by training stage, forming a "captioning-quality pyramid" that corresponds to the data pyramid: the massive-scale low-resolution (256px) stage is captioned with the open-source **LLaVA-Video**; the curated 5M data in the high-resolution (768px) stage is instead recaptioned with the stronger **Qwen 2.5 Max** (a closed-source API model), the reasoning being "to obtain more accurate captions with better semantic alignment." This "coarse captioning for the base + refined captioning for the top" tiered captioning is an important part of its low-cost strategy.
[Open-Sora Plan v1.3] The primary model is **Qwen2-VL-7B-Instruct**; the Panda70M portion directly reuses its official public captions; the stock-footage portion uses **ShareGPT4Video**'s annotations; the VIDAL portion retains its original multi-model refined captions from OFA / mPLUG-Owl / ChatGPT. In addition, a separate **prompt refiner** was trained: LoRA fine-tuning on a **LLaMA-3.1-8B-Instruct** base (rank 64, batch 32, 1 epoch, trained in 30 minutes on a single H100).

### Caption density and degree of structuring (short/long/dense description, structured fields such as camera motion, style tags) ⚠️

Both projects follow the **long, dense, structured caption** route, and both write their structured-field designs explicitly into prompt templates that can be directly reused:
[Open-Sora 2.0] Requires LLaVA-Video to cover **six aspects**: 1) main subjects; 2) subjects' actions; 3) background and environment; 4) lighting condition and atmosphere; 5) camera movement; 6) video style. Length statistics (Figure 3): **more than 70% of video captions exceed 75 words in length**. In addition, the **motion score is directly appended to the end of the caption text**, allowing motion magnitude to be controlled via text at inference time.
[Open-Sora 1.x] Adopts a concatenated "description + explicit condition tags" structure: after the natural-language description generated by PLLaVA, three types of scalar/enumerated conditions — **aesthetic score, motion score, camera motion** — are appended, in the form `aesthetic score: 5.5, motion score: 10, camera motion: pan left`. This "score condition" mechanism is a hallmark design of Open-Sora 1.2, allowing the same model to control image quality and camera work via text at inference time without needing an extra conditioning channel.
[Open-Sora Plan v1.3] The prompt requires the model to describe objects, scenes, people, and camera motion **in chronological order**; post-processing uses a table of 28 common opening phrases to strip redundant prefixes such as "The video shows." Samples with aesthetic score > 6.25 automatically get "A high-aesthetic scene" prepended.
[Explicit handling of the training-inference caption distribution mismatch (unique to Open-Sora Plan)] The authors note that training captions are dense and lengthy while users' actual prompts are often under 10 words; to address this, they specifically built a **19,500-caption refiner training set**, covering four length/style categories: 11,000 short user-style captions from COCO, 5,000 tag-style captions from DiffusionDB, 3,000 medium-length LLM captions from JourneyDB, and 500 ultra-long, surreal, GPT-generated captions from official Sora/Vidu/Pika/Veo demos; ChatGPT is used to uniformly rewrite these into a target format of "subject description + action + scene description (+ optional cinematography and atmosphere)," and then LLaMA-3.1-8B is LoRA fine-tuned as an inference-time prompt expander. This is the most systematically handled case of "caption distribution mismatch" in this research, with fully disclosed training-data composition.

### Joint audio-video caption structure (whether both visual and auditory tracks are covered, whether split into independent fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Not applicable. Neither project generates audio; captions cover only the visual track (subject/action/background/lighting/camerawork/style), with no auditory-track description field, and no LTX-2-style full soundscape description or Foley-Omni-style three-field split design.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion)

Not applicable. The data pipeline does not process audio tracks at all; there is no ASR transcription and no speaker identity/language/accent/emotion annotation.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

Annotation depth is relatively shallow, with only two-dimensional motion-level structured information and no three-dimensional geometric annotation:
- Present: **camera-motion labels** — Open-Sora 1.x classifies camera motion (pan left, zoom in, etc.) based on UniMatch optical flow, labeling only high-confidence clips and writing the result into the caption, with the module separately open-sourced under tools; Open-Sora 2.0 and Open-Sora Plan instead treat camera motion as a caption description field generated in natural language by the VLM, not an independent structured field.
- Present: **scalar quality conditions** — aesthetic score, motion score (VMAF/optical flow/LPIPS), OCR text-region count, and DOVER technical score are all stored as structured columns in the meta CSV, with some spliced into the caption as controllable conditions.
- Absent: camera intrinsics/extrinsics, depth maps, 3D point tracks, optical-flow fields used directly as conditioning input, explicit object-state/action labels, segmentation masks, etc.
- Open-Sora Plan v1.3 trained an independent **Structure Controller**, extracting three types of structural signal — **canny edges, depth maps, and sketches** — from the data as control conditions (20k steps, 8 NPU cards); this is the only place involving depth-information annotation, but its purpose is ControlNet-style conditional control rather than the annotation schema of the main training data.

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV)

Neither project **uses model-synthesized video data**; all training videos come from genuinely collected public datasets. The only "constructed" data that exists falls into two categories, neither of which is video synthesis:
1. **Mask-constructed multi-task training pairs (Open-Sora Plan v1.3 Image Controller)**: on the same batch of real videos, **different frame-masking patterns** construct four task-sample types — T2V, I2V (first-frame conditioning), Transition (first-and-last-frame conditioning), and Continuation (video continuation); stage 1's 50M samples are divided into 7 progressive sub-steps, with the frame-retention rate progressively lowered from 50% → 25% → 12.5%, and the task mixture progressively tilting toward I2V (40%) and Transition (40%); stage 2's 15M high-quality data continues this mixture. This is a typical way of deriving multi-task supervision signal from a single set of real data via masking, and does not involve a generative model.
2. **Caption-level synthesis (Open-Sora Plan's prompt refiner)**: ChatGPT is used to rewrite 19,500 real-sourced captions into unified "short prompt → long prompt" pairs — a text-side synthesis.
In addition, Open-Sora 2.0 builds a text-to-image-to-video pipeline on the inference side (first text-to-image with FLUX, then image-to-video), but this is an inference strategy, not training-data synthesis.

### Degree of human involvement (human annotation, human quality inspection, model-prescreen + human review)

The degree of human involvement is very low, used mainly for **threshold calibration and sample verification** rather than large-scale annotation or review:
- [Open-Sora Plan] The only explicitly recorded human step is: to verify the reliability of the LPIPS motion-filtering threshold (0.001-0.3), **2,000 videos were manually inspected**; the method was applied at full scale only after confirming its accuracy was adequate. This is a low-cost paradigm of using human effort to "verify an automated rule."
- [Open-Sora series] The documentation records no human annotation or human quality-inspection step at all; captions are generated entirely by VLMs with no human review; the tools/caption documentation instead explicitly states that "manually annotating videos is expensive and time-consuming, so powerful image/video description models are used to generate captions" — an explicit statement of actively forgoing human annotation.
- Human involvement is more evident on the **evaluation side**: Open-Sora 2.0 uses 100 prompts for a human preference evaluation (comparing against HunyuanVideo, Runway Gen-3 Alpha, Step-Video-T2V, and Luma Ray2), scoring win rates across three dimensions — visual quality, prompt adherence, and motion quality.
- Both projects are, in essence, "a fully automated pipeline plus a very small amount of human spot-check calibration," which is also the precondition for their ability to process tens of millions of videos at extremely low cost.

## Audio-Video Alignment

### Audio-video synchronization detection methods (lip sync, event alignment)

Not applicable. There is no audio modality, so no audio-video synchronization detection step exists.

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 ∧ LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5)

Not applicable. Neither SyncNet, AV-align, nor any synchronization metric is used.

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching treated as two independent filtering conditions)

Not applicable. There is no audio modality. (If forced into an analogy, visual-text semantic matching is handled by Open-Sora 1.x's CLIP matching score, but this is unrelated to audio-video synchronization.)

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, removal of no-audio-track clips, removal of off-screen sound sources, background-music separation)

Not applicable. The pipeline does not read audio tracks at all; transcoding uniformly produces H.264 video streams, with no SNR, silence detection, missing-audio-track removal, or voice/background-music separation of any kind.

### Classification and separate handling strategies for speech/sound effects/music

Not applicable. There is no classification or differentiated handling of speech/sound effects/music.

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

Both projects adopt a clear multi-stage progressive curriculum in which **data stratification is tightly bound to training stage** — the core of their low-cost strategy:
[Open-Sora 2.0 (three stages, corresponding one-to-one with the data pyramid)] Stage 1: 70M videos, 256px, pure T2V, 85k iterations, 224 cards; Stage 2: 10M videos, 256px, T2V+I2V mixed, 13k iterations, 192 cards; Stage 3: 5M videos, 768px, T2V+I2V, 13k iterations, 192 cards. The curriculum dimension is a simultaneous tightening of "resolution (256→768) × data quality (loose→strict) × task (T2V→adding I2V)." The report emphasizes that the low-resolution stage is for low-cost learning of motion patterns while the high-resolution stage is for improving perceptual quality, and specifically notes that **image-to-video (I2V) fine-tuning is more compute-efficient than pure text-to-video** — using I2V to carry the high-resolution stage is a key decision in compressing cost.
[Open-Sora 1.2] Stage 1, 30k steps: Webvid-10M, 240p/360p, 2-16 seconds; Stage 2, 23k steps: the aesthetic-score > 4.5 subset of Panda-70M (41k hours), 360p/480p; Stage 3, 15k steps: about 2M high-quality clips (5k hours), 720p/1080p, combined with 25% masked training. 68k steps total.
[Open-Sora Plan v1.3 (T2V)] Stage 1, 100k steps: pure images at 1×320×320 (SAM), used to smoothly migrate 3D dense attention to sparse attention; Stage 2, 300k steps: joint image-video, up to 93×320×320, using **unfiltered** Panda70M (about 19M); Stage 3, 30k steps: using only **filtered** Panda70M, fixed at 93×352×640, learning rate decreasing from 2e-5 to 1e-5. The curriculum dimension is "image → video, low-res → high-res, coarse data → refined data."
[Open-Sora Plan v1.5] 4 image stages (256² → 384² → 288×512) + 5 video stages (57 frames → 121×576×1024), with the final stage using a high-quality subset; the entire run is trained on Ascend 910B NPUs.
Common paradigm: **a low-quality large-data base plus a high-quality small-data finish**, with "filtering strictness" itself serving as an explicit dimension of curriculum scheduling (the contrast is most explicit in Open-Sora Plan, where stage 2 uses unfiltered data and stage 3 switches to filtered data).

### How data mixture changes across training stages (pretraining/annealing/SFT high-quality subset)

[Open-Sora 2.0] The data-volume ratio across stages is 70M : 10M : 5M (14 : 2 : 1); the latter two stages simultaneously introduce the I2V task and mix it with T2V training (the report gives no specific T2V/I2V mixture figure). There is no separately named annealing stage, but stage 3 (5M high-quality data + 768px) effectively plays the role of annealing/SFT.
[Open-Sora 1.2] Stage 3 applies **25% masked training** (mask ratio) to the high-quality data, giving the model both image-to-video and video-continuation capability during the high-resolution fine-tuning stage.
[Open-Sora Plan v1.3] On the T2V side, it's a two-part scheme of "unfiltered data (stage 2) → filtered data (stage 3)"; the I2V-side task-mixture scheduling is very fine-grained: stage 1's 50M samples are split into 7 progressive sub-steps, transitioning gradually from continuation/random masking (frame-retention rate 50% → 25% → 12.5%) to being dominated by I2V (40%) and Transition (40%); stage 2's 15M high-quality data maintains this mixture and raises the resolution to 93×640×640. This is the **most finely described open-source case of task-mixture change across stages** in this research.
[Open-Sora Plan v1.5] The image side uses the full 1.1B unfiltered, the video side uses 40M filtered data, and the final stage switches to a high-quality subset.

### Post-training data (scale and selection criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

Neither project has **post-training in the modern sense (RLHF/DPO/reward model)** — there is no preference-pair data, no reward-model training set, and no human-preference alignment. What is called "post-training" is actually just continued pretraining/fine-tuning on high-quality data:
- Open-Sora 2.0: stage 3's 5M curated high-resolution data plus Qwen2.5-Max recaptioning is its SFT equivalent; the selection criteria are the strictest threshold combination at the top of the data pyramid (specific threshold values not given).
- Open-Sora 1.2: stage 3's 2M-clip / 5k-hour high-quality set, sourced from MiraData, Vript (GPT-captioned), and other free footage sites (PLLaVA-captioned).
- Open-Sora Plan: the T2V stage 3 filtered Panda70M (about 19M / 27% retained) plus the I2V stage 2's 15M high-quality set.
- The only "independent post-training module" is Open-Sora Plan's **prompt refiner**: 19,500 captions, LLaMA-3.1-8B LoRA (rank 64, batch 32, 1 epoch, 30 minutes on a single H100), but this fine-tunes the text model rather than the video model.
- Human preference data is used only for **evaluation** (Open-Sora 2.0's 100-prompt human win-rate evaluation) and is not fed back into training.

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

[Framework] Neither uses a mature data-processing framework such as **NeMo Curator or Data-Juicer**; instead both use self-built lightweight pipelines: Open-Sora's is a combination of CSV meta files plus torchrun multi-card parallel scripts (`torchrun --nproc_per_node 8 -m tools.scoring.aesthetic.inference meta.csv --bs 1024`), with each step producing a new column, merged and filtered by `tools.datasets.datautil`; the data flow is centered on "a continually widening meta CSV," an extremely minimal, easy-to-reproduce design. Open-Sora Plan has not published engineering-implementation details of its data processing.
[Throughput (the only measured figure officially given by Open-Sora)] Aesthetic scoring runs at **about 1,000 videos/second on a single H800**, with support for linear multi-card scaling; no throughput figures are given for heavier modules such as OCR or optical flow. This is one of the few pieces of public material in this research that gives a concrete single-card data-processing throughput figure.
[Cost — the most iconic data point in this entry] Open-Sora 2.0's stage-by-stage training cost is fully disclosed: Stage 1, 224 cards × 85k iterations = **$107.5k**; Stage 2, 192 cards × 13k iterations = **$18.4k**; Stage 3, 192 cards × 13k iterations = **$73.7k**; **total $199.6k (about $200,000)**, to train an 11B model. Open-Sora 1.2's training consumption is 68k steps total, about **35,000 H100 GPU-hours**. Note: these cost figures cover **training only**; neither project **discloses the compute cost of the data-processing step itself** — given that aesthetics, optical flow/VMAF, OCR, blur detection, and VLM captioning must all be run over tens of millions of videos (VLM captioning in particular is extremely expensive), the data-side cost is very likely a hidden major component, and the "$200k to train a commercial-level model" framing should be understood with this in mind. [Uncertain]

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

Neither project's report contains a **strict data-ablation experiment** (no side-by-side table of "data strategy A vs. data strategy B under identical architecture and compute"), the main weakness of their technical reports relative to academic norms. The available quantitative/qualitative evidence is as follows:
[Filtering-strictness dimension] Open-Sora Plan v1.3 provides the design evidence closest to an ablation: training stage 2 uses **unfiltered** Panda70M while stage 3 switches to **filtered** data; the authors report that stage 2 nearly converges around 100k steps, after which stage 3's filtered data (only 30k steps, learning rate reduced to 1e-5) achieves a quality leap — indirectly indicating that "a large amount of coarse data is responsible for learning the distribution, while a small amount of refined data is responsible for raising quality." However, no metric comparison between the two settings is given. The paper also gives reasoned justification for its threshold choices (aesthetic 4.75 is chosen because it also has the effect of filtering text; the LPIPS upper bound of 0.3 is because exceeding it produces jitter/flicker; these conclusions were verified via 2,000 manual spot-checks) — this is "threshold-rationale argumentation" rather than ablation.
[Caption density/style dimension] Open-Sora Plan explicitly argues that the distribution mismatch between training captions (dense, long text) and user prompts (<10 words) harms generation quality, and builds the prompt refiner motivated by this — a qualitative argument about the impact of caption style, but again with no quantitative comparison of the refiner on vs. off. Open-Sora 2.0 switches the captioning model from LLaVA-Video to Qwen2.5-Max in the high-resolution stage, reasoning that it is "more accurate, with better semantic alignment," again with no controlled experiment.
[Data-mixture dimension] No mixture ablation of any kind.
[Overall effectiveness metrics] Open-Sora 2.0's VBench results show its gap to Sora narrowing from Open-Sora 1.2's **4.52% to 0.69%**, and it surpasses CogVideoX1.5-5B and HunyuanVideo; in the 100-prompt human evaluation, it beats HunyuanVideo, Runway Gen-3 Alpha, Step-Video-T2V, and Luma Ray2 on at least two of the three dimensions of visual quality/prompt adherence/motion quality. But these are end-to-end results that cannot be attributed to any single data-strategy factor. [Uncertain]

### Evidence on quality vs. quantity (cases where small, precise data beats large, messy data)

This is the most persuasive body of overall evidence in the Open-Sora series, despite the lack of a strict controlled experiment:
1. **Open-Sora 2.0, with $199.6k and 11B parameters, using only 5M videos in the high-resolution stage, achieves VBench and human-preference levels comparable to the 11B HunyuanVideo and the 30B Step-Video-T2V**, at a training cost 5-10x lower than peer-tier models. Its core mechanism is precisely "hierarchical data pyramid + tiered captioning + progressive curriculum": spending the vast majority of compute on loosely filtered low-resolution data, and using only the most strictly filtered 5M data (7% of the base tier's 70M) for the high-resolution finish.
2. **After cutting Panda70M down to a 27% retention rate** (or 42% by the paper's funnel-table figure), **Open-Sora Plan v1.3** achieves a significant quality improvement with only 30k steps of fine-tuning, and v1.5 reaches a level comparable to HunyuanVideo using 40M videos and an 8B model — likewise a route of "filter aggressively, train little."
3. **Open-Sora 1.2** filters 30M clips / 80k hours down to 2M clips / 5k hours (about 6.7%) as high-resolution-stage data.
All three versions consistently keep their final high-quality subset at roughly the **7% order of magnitude** of the original pool, and this ratio is a converging empirical value across projects and versions — in itself strong empirical evidence for "quality over quantity." Counter-note: there is no control for "what would happen if stage 3 used 5M randomly sampled, unfiltered data instead," so strictly speaking this is **engineering-practice evidence rather than scientific evidence**.

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g. VABench's seven major categories) ⚠️

Neither project's training data has **established a domain-taxonomy system**, so there is no meaningful alignment with any evaluation-benchmark taxonomy to speak of. Evaluation primarily uses **VBench** (Open-Sora 2.0's report Table 13 gives complete per-dimension VBench scores, with "the gap to Sora narrowing from 4.52% to 0.69%" as the headline metric) and a self-built 100-prompt human preference evaluation (three dimensions: visual quality / prompt adherence / motion quality). VBench's sixteen sub-dimensions (subject consistency, background consistency, temporal flickering, motion smoothness, dynamic degree, aesthetic quality, imaging quality, object class, multiple objects, human action, color, spatial relationship, scene, appearance style, temporal style, overall consistency) have a **partial, indirect correspondence** with the training data's filtering dimensions: aesthetic quality ↔ LAION aesthetic-score filtering, imaging quality ↔ DOVER/Laplacian/bpp filtering, dynamic degree and motion smoothness ↔ bidirectional optical-flow/VMAF/LPIPS motion filtering, temporal flickering ↔ removal of flickering clips via the LPIPS upper bound. In other words, filter design is in fact built around VBench's **low-level image-quality and motion dimensions**, but there is no data-side mixture or coverage guarantee for VBench's **semantic-category dimensions** (object class, multiple objects, human action, spatial relationship, scene) — the same issue as the previously noted absence of domain_distribution, viewed from another angle. There is no relationship whatsoever to AV benchmarks such as VABench (no audio capability). [Uncertain]

## Other Information

### summary_note

Core value proposition: Open-Sora and Open-Sora Plan are the two samples with the **highest reproducibility of data-processing methodology** in this research, forming a perfect complement to closed-source models such as Sora 2 / Veo 3 that are "most capable but disclose nothing." Specific reusable takeaways: (1) The tools/ directory in the Open-Sora v1.2.0 branch provides a **code-level, threshold-annotated, throughput-annotated complete video data pipeline** (PySceneDetect shot splitting → CLIP+MLP aesthetics → UniMatch optical flow → DBNet++ OCR → PLLaVA-13B captioning → CLIP matching → camera-motion detection → datautil filtering), the most direct starting point for building one's own pipeline; (2) the Open-Sora Plan paper provides a **stage-by-stage retention-rate table rarely seen across the whole industry** (100%→97%→89%→89%→49%→44%→42%, with the aesthetic-filtering level alone eliminating 40 percentage points), plus a 27% final retention-rate figure highly consistent with Apollo's; (3) Open-Sora 2.0's **hierarchical data pyramid + tiered captioning (open-source LLaVA-Video for low resolution, Qwen2.5-Max for high resolution) + stage-by-stage cost table ($107.5k/$18.4k/$73.7k = $199.6k)** is a complete disclosure of a low-cost training recipe; (4) the shared design across both of "appending aesthetic score/motion score/camera motion into the caption as controllable conditions" is a widely reused engineering pattern.
Main limitations (to note when citing): (1) The entire series **does not support audio**; the 8 audio-video-alignment-related fields are entirely not applicable, and it cannot serve as a data-processing reference for AV models; (2) there is **no deduplication, no safety filtering, and no VLM-as-judge** — filtering relies entirely on shallow scalar scorers plus thresholds, representing the 2024-2025 technical water mark, with a clear generational gap relative to the 2026 trend toward "large-model semantic judgment"; (3) there is **no domain mixture control or concept balancing** — data distribution is passively inherited from the public datasets; (4) there is **no strict data-ablation experiment** — "quality over quantity" is engineering-practice evidence rather than controlled-experiment evidence; (5) Open-Sora 2.0's main branch **has removed the data-processing code**, and the new pipeline described in the report (PaddleOCR / VMAF / Laplacian) has no corresponding open-source implementation — strictly speaking, the assessment of "complete open-source data-processing code" applies only to the 1.x versions; (6) data sources are entirely public datasets (Panda-70M / Webvid / LAION, etc.) whose copyright status is disputed, and the $200k cost figure does not include data-processing or captioning compute.

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

- openness
- data_scale
- data_sources
- provenance_licensing
- domain_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- data_infra_throughput
- data_ablation
- benchmark_taxonomy_alignment
