# Joint Audio-Video Generation Models: Data and Training Playbook (Comprehensive Best Practices)

> Distilled from a horizontal comparison of this wiki's 41 entries, prioritizing two kinds of sources: **the latest 2025H2–2026 work** (Seedance 1.5pro/2.0, Kling 3.0 Omni, Sora 2, Veo 3.1, LTX-2, MOVA, Apollo, Vidu S1, CineDance, Wan 2.5+, etc.) and **work with the most thorough disclosure / with quantitative ablation evidence** (Movie Gen, Allegro, Goku, HunyuanVideo, Open-Sora 2.0, Cosmos-Predict2.5, Koala-36M, MiraData, Data-Juicer, Wan 2.1, etc.). Four additional horizontal topic entries provide cross-work synthesis: the **evaluation benchmark collection** (VABench/AVBench/AV-SyncBench/PhyAVBench/Omni-Judge), the **post-training data topic**, the **captioner ecosystem topic**, the **pretraining dataset lineage** (Panda-70M→InternVid→Koala-36M→LVD-2M→OpenVid-1M→MiraData→UltraVideo), and the **geometric/structured annotation datasets** (SceneScribe-1M/SpatialVID/WildWorld/Action100M). Each practice item is annotated with its source in parentheses.
>
> Generated on: 2026-07-29 · [← Back to home](index.md)

---

## 0. Overview of the Approach (TL;DR)

| Stage | Recommended practice | Key numeric anchors |
|------|----------|--------------|
| Raw pool | Collection at tens-of-millions-of-hours scale, videos without audio tracks removed upfront | HunyuanVideo 1.5 >10M h; Cosmos 35M h |
| Cleaning funnel | 14-level cascade: cheap rules first, VLM final review last | End-to-end retention rate: AV joint ~25%, strict tier for pure video 1–5% |
| AV alignment | Temporal (Synchformer/SyncNet) and semantic (ImageBind) **split into two separate metrics** | SyncNet \|offset\|≤3 ∧ conf>1.5 has become the de facto standard |
| Captioning | Audio-perceptive models + dense structured captions + multi-granularity mixed training | 100–250 words, short/medium/long tiers randomly sampled |
| Visual curriculum | T2I foundation → 256p→480p→720p pyramid, volume shrinks as quality rises | Data volume shrinks stage by stage: 200M→100M→1M (HunyuanVideo 1.5) |
| Modal curriculum | Massive pure-audio pretraining of the audio tower first → small amount of paired data for joint adaptation | Skipping this causes audio collapse: WER worsens 4× (Foley-Omni) |
| SFT | Manual curation + caption re-annotation, 0.4–20% of pretraining volume | HunyuanVideo 1M; NAVA 1.07%; CogVideoX 20% |
| Preference alignment | Online RL with GRPO + multi-dimensional reward (including AV-specific RM) | Preference-pair scale 10⁴–10⁵ (SkyReels-V2 30K, JavisDiT++ 25K) |
| Infrastructure | NeMo Curator (GPU) or Data-Juicer (CPU) + bucketed sharding | 2000×H100 processes ~1M hours of 720p video in one day |

Core beliefs (backed by strong evidence, see §9): **the quality of data processing itself is the single biggest lever** — with the same source material and the same architecture, improving processing alone yields VBench +9.67 points (Koala-36M); **repeating high-quality data 6 times beats low-quality data with 8× the compute** (Data-Juicer). A supplementary corollary: **video quality and caption quality are two orthogonal levers for improvement** — swapping only the caption substantially improves dynamics/temporal consistency/text alignment while not improving visual fidelity at all (MiraData), so the two should each be invested in independently rather than substituted for one another.

---

## 1. Data Scale and Collection Targets

**Pretraining (visual backbone)**: raw pool at tens-of-millions-of-hours scale, retained through the funnel at 1–5% to yield hundreds of millions of clips (HunyuanVideo 1.5: >10M hours → 800M clips; Cosmos-Predict2.5: 35M hours → 6B candidates → 200M clips, retention rate about 4%).

**Audio-video paired corpus**: can be one order of magnitude smaller, starting at tens-of-thousands-of-hours / tens-of-millions-of-pairs scale (MOVA three stages 61.5k→37.6k→11k hours; ALIVE joint training 11M pairs; Movie Gen audio pretraining at million-hour scale).

**Standalone audio-tower pretraining**: hundreds of thousands of hours of pure audio, predominantly speech (ALIVE 700k hours of transcribed speech; Ovi, SkyReels-V4 hundreds of thousands of hours; Unison 130k hours).

**SFT**: million-scale manually curated (HunyuanVideo ~1M; SkyReels-V4 5M→1M two stages). Across 15+ works, the curation-ratio range relative to pretraining volume is **0.4%–20%** (Allegro 0.4%, NAVA 1.07%, Cosmos ~10%, Goku 12.5%, CogVideoX/SkyReels-V4 stage two 20%), with absolute scale of 10⁶–10⁷; preference-alignment data is a further 2–3 orders of magnitude smaller, at 10⁴–10⁵ pairs (post-training data topic).

**Person/dialogue-specific corpora**: two-person interaction datasets can reach ten-thousand-hour scale — SpeakerVid-5M goes from 153K source videos/64,386 hours to 5.2M clips/8,743 hours (end-to-end duration retention rate about 13.6%), further cutting down to 571K clips/1,368 hours for SFT (about 2.1% relative to the original material). Note that the primary loss here is not quality filtering but **structural loss** (retaining only the two main speakers, retaining only clear speaking/listening segments, single-person cropping loses picture content), so this retention rate should not be directly compared to MOVA's 26.39%/Apollo's 27% (SpeakerVid-5M).

**Staged pyramid**: data volume shrinks progressively with resolution/quality (HunyuanVideo 1.5: 480p 200M → 720p 100M → CT 1M; Goku: 480p 36M → 720p 24M → 1080p 7M).

---

## 2. Duration Splitting and Resolution Strategy

**Splitting**: TransNetV2 scene splitting (NeMo measured F1=0.967, outperforming PySceneDetect/AutoShot) or dual-tool complementary approaches (LongCat, SkyReels), followed by embedding-similarity stitching to prevent over-splitting (NeMo, Motif, Panda-70M), dropping 3–10 frames at head/tail to remove transition residue (Step-Video, Allegro). Two lightweight alternatives can be directly adopted: **Koala-36M's Color-Struct SVM** (open source, with fitted coefficients given as `4.61480465×bgr_sim + 3.75211168×canny_sim − 5.485968377` + a 3σ temporal threshold, accuracy/recall of 0.7741/0.9395 far exceeding PySceneDetect's 0.4574/0.4146, and faster above 1080p); **LVD-2M's low-sampling-rate trick** (ContentDetector threshold of 50 + 0.5fps sampling, using "any significant change within 2 seconds counts as a cut" to catch fade-in/fade-out misses at zero extra cost). When pursuing long shots, use a "over-split then stitch" paradigm: deliberately lower the threshold to over-split, then stitch back together via pairwise consistency voting within groups using VLM and ImageBind+DINOv2 (MiraData — the only viable path to obtaining 72-second long shots).

**AV-specific hard constraint**: cut points must be speech-aware — drive the split window jointly with VAD and scene cut points, never cutting a sentence in half (MOVA fixed 8.05s window = 193 frames@24fps; same for Vidu S1).

**Main training duration** 5–15 seconds (HunyuanVideo 1.5 uniformly 2–10s; Movie Gen 4–16s, with >50% falling in 15–16s). Training windows extended to the 15-second tier in 2026 (Seedance 2.0 direct output 4–15s, Kling 3.0 targeting 15s). Two parallel approaches coexist — fixed duration (MOVA/Ovi/HunyuanVideo-Foley) and duration bucketing (Movie Gen five buckets, SkyReels-V2 dual-axis bucketing) — fixed duration is simpler to implement, bucketing yields higher data utilization.

**Resolution**: a progressive curriculum of 256p→480p→720p is consensus (Cosmos, HunyuanVideo 1.5, MOVA — the latter's ablation proves that aligning at low resolution first before upscaling is effective), with a small-scale high-resolution polish at the tail end (Cosmos 4K cooldown, 388K clips). Native high resolution is significantly better than low resolution + super-resolution (UltraVideo: VQAA 73.46 vs. 69.26) — **resolution is the one dimension that cannot be compensated for by later-stage filtering**, fidelity must be preserved at collection time. 1080p+ can be outsourced to an independent super-resolution module to cut cost (HunyuanVideo 1.5).

**Aspect ratio**: the only quantitative mixture ratio comes from Movie Gen — pretraining 60% landscape + 40% portrait, 80/20 at the high-resolution stage, five AR buckets; a simpler route is normalizing to two aspect ratios (16:9/9:16) via padding (MOVA), or equal-area normalization (Ovi's fixed pixel product, which naturally supports arbitrary AR).

---

## 3. Cleaning Funnel (Visual Side)

**General principle**: a cascade review with increasing compute cost — cheap rules do coarse filtering first → dedicated small models score in the middle → expensive VLMs do final review and labeling at the end (Cosmos, Allegro, MAGI-1, MOVA, Apollo all agree). The trend is to have the VLM's "scoring" and "labeling" share a single forward pass (MAGI-1/Motif), structurally eliminating drift between filtering and conditioning.

**Recommended cascade order** (synthesized across works):

1. **Audio-track existence check** (a one-shot veto for AV models, placed first: MOVA, HunyuanVideo-Foley, UniVerse-1)
2. **Hard-metric prescreen**: duration ≥2s, resolution ≥720p (Ovi >720×720), bitrate ≥500kbps–1Mbps (Goku, Foley-Omni, to guard against "fake HD"), fps ≥23
3. **MD5/source-ID exact deduplication**
4. **TransNetV2 splitting + stitching + head/tail frame removal** (speech data jointly windowed with VAD)
5. **Black-bar/subtitle/watermark detection and cropping** — prefer cropping over discarding (HunyuanVideo 1.5: discard only if <60% remains after cropping; ffmpeg cropdetect for black bars)
6. **Aesthetics + technical quality**: LAION aesthetic score starting at ≥4.0–4.75, progressively tightened to 5.3 (Movie Gen 4.0 → Allegro 4.8→5.3); DOVER technical score as a backstop; OCR text area ≤1–2% (Goku)
7. **Bilateral optical-flow motion filtering**: filters out both stillness and jitter (RAFT/UniMatch; Goku 480p tier 0.3–20; Movie Gen uses "shots per second ≥0.85" as a jitter proxy). A more mature form is **six-tier grading + differentiated sampling rates rather than binary thresholds**: optimal-motion and moderate-motion samples are retained, static video (interviews) and camera-dominated motion (aerial footage) are **downsampled rather than deleted** to preserve concept coverage, and only low-quality motion (too many subjects/severe occlusion) and jittery shots are truly excluded (Wan 2.1)
   - Absolute threshold reference (the one set that gives full numeric values ready to adopt directly): PaddleOCR text area >2%, black-bar-region mean <3, overexposed-pixel ratio >12%, RGB variance <1.2, with a unified "discard whole clip if bad-frame rate >5%" rule (UltraVideo)
8. **Audio quality filtering** (see §4)
9. **AV alignment filtering** (see §4)
10. **NSFW/safety classifier** (Seedance's dedicated classifier; lightweight option: LAION CLIP NSFW)
11. **SSCD semantic deduplication**: embedding clustering → intra-cluster comparison → retain the highest-resolution version (Cosmos/NeMo/Seedance 1.0; NeMo measured a removal rate of about 30%)
12. **VLM final review** (Qwen2.5-VL chained review at the end: Cosmos; sharing a forward pass with labeling: MAGI-1)
13. **MLLM dense labeling + independent-channel reverse verification** (OmniHuman uses deterministic tracking/ASR outputs to reverse-check MLLM hallucinations)
14. **Concept-balanced resampling + bucketed sharding** (see §6)

**Reference implementation · Wan 2.1's four-step funnel** (one of the most completely disclosed pure-video cleaning blueprints on the open-source side): step one, "basic dimensions," runs 9 lightweight detectors in parallel (OCR coverage, LAION-5B aesthetics, NSFW, watermark/logo, black bars, overexposure, synthetic-image detection, blur, duration and resolution thresholds); **this single tier alone eliminates about 50%**; step two, "visual quality," splits into "clustering + scoring" — first clustering into **100 clusters with per-cluster quota sampling**, then scoring the full dataset with an expert model trained on human 1–5 ratings, where the explicit purpose of the quota is to prevent "small but important" long-tail data from being wiped out entirely by a global threshold; step three, "motion quality," applies six-tier differentiated treatment; step four is post-training curation (top 20% for images via the expert model, videos balanced across 12 major categories). Finally the full dataset is relabeled (Wan 2.1). Its **leverage-style annotation paradigm** is reused repeatedly across quality scoring, camera motion, and artifact detection: "manually label a small seed set → train an expert small model → auto-label the full dataset."

**Retention-rate anchors**: AV joint funnel end-to-end about 1/4 (Apollo 27%, MOVA 26.39%); pure-video strict tier 1–5% (Cosmos 4%, Movie Gen high-resolution tier <1%, where resolution, aspect ratio, and OCR each cut 72–75%); SFT premium rate a further two orders of magnitude lower (NAVA 15M→160K, about 1%). Open-source datasets give clearer tiered empirical values: no quality filtering ≈ 100% (Panda-70M, actually an 18.7× expansion from 3.8M source videos → 70.8M clips) → standard quality filtering ≈ 70–75% (Koala-36M 48M→36M, UltraVideo statistical-layer 74–76%) → **stacking long-shot + high-dynamics + no-cuts constraints drops retention sharply to about 1%** (LVD-2M 220M→2M, i.e., 0.91%; MiraData retains only 0.2% of HD-VILA-100M) — the cost of this set of constraints is 2–3 orders of magnitude of data loss, in exchange for a long-shot purity rate of 77.5% (vs. Panda-70M's 50.0%) and a "highly dynamic" share of 30.0% (vs. 7.5%) (pretraining dataset lineage).

**Warning · upstream dataset self-evaluation**: Panda-70M's own supplementary desirability annotations show that about 19.5% of the full set is still undesirable (low desirability score 5.28%, static foreground 6.82%, picture-in-picture 5.03%, screen recording 1.13%); LVD-2M's human evaluation shows 50% of clips contain scene cuts and 25% are judged "not dynamic" — **using a public dataset directly does not exempt you from cleaning**, which is exactly why VidGen-1M/OpenVid-1M/Koala-36M were all founded on "redoing Panda-70M" (pretraining dataset lineage).

**Engineering philosophy divergences** (pick as needed): multi-threshold cascading (mainstream) vs. a single learned composite score (Koala-36M's VTSS, argued on the grounds that sub-metrics are non-orthogonal); hard filtering vs. full-set scoring stored as metadata with downstream tightening as needed (CineDance — recommended for large pools, flexibly supporting curriculum stratification); discarding low-quality data vs. loss isolation (UniVerse-1's LQLS: loss applied to low-quality data only at diffusion t>800; removing it drops ID consistency from 0.89→0.78 — "restricting the scope of effect" beats outright discarding).

---

## 4. Audio Filtering and Audio-Video Alignment

**Recommended audio-side filtering chain**:

1. Remove clips with no audio track/decode failure → 2. Format normalization (44.1kHz) → 3. Volume ≥−60dB (Ovi/SkyReels-V4) + tiered silence-ratio thresholds (**speech data <0.2 / sound-effects data <0.8** — HunyuanVideo-Foley deliberately loosens this to preserve "silence + one door-slam" style Foley samples) → 4. Bandwidth verification: spectral detection of effective sample rate >32kHz, removing upsampled fake-HD audio (HunyuanVideo-Foley) → 5. Perceptual quality: **Audiobox-Aesthetics has replaced SNR as the de facto standard** (MOVA: PQ>5.0 / CU>4.5 / CE>2.5) → 6. Four-way audio classification (speech/singing/sound effects/music, SkyReels-V4 uses Qwen3-Omni) → 7. Route to alignment (below) → 8. Store all scores in metadata (CineDance).

**Three ASR diagnostic thresholds specific to the speech subset** (directly reusable): Whisper transcription confidence <−1.5 removed (HQ subset requires >−1), no-speech probability >0.8 removed, compression ratio >2.5 removed (used to identify repetitive-degeneration/hallucinated output), plus language-mismatch detection (SpeakerVid-5M). This set of thresholds extends ASR from "producing text only" to "doubling as an audio-quality discriminator," at essentially zero extra cost.

**A quality-control template reusable for any step requiring human review**: **"high-recall coarse filtering with a commercial closed-source multimodal LLM + N=5 annotator pool, ≥3 independent cross-reviews per sample."** AV-SyncBench uses Gemini 3 Flash as an automated first-pass filter to remove two categories — "off-screen sound sources" and "obvious audio-visual mismatch" — followed by 5 annotators, each sample independently reviewed by at least 3 for primary-sound-source visibility. Using a lightweight model like Flash for recall and human effort for precision is a cost-effectiveness trade-off rarely seen embedded directly at the first stage of a public pipeline (AV-SyncBench).

**Splitting temporal vs. semantic sync is a strong consensus** (since 2025H2): Synchformer/AV-align handles temporal sync, ImageBind handles semantic sync, their blind spots complement each other. Combination logic:

- **General soundscape data uses an OR gate** (MOVA: ImageBind≥0.2 OR DeSync≤0.5) — ambient sound without a sharp onset gets wrongly killed by the temporal detector, fast-action sound effects get wrongly killed by the semantic detector; an OR gate on both sides preserves diversity
- **Curated subsets use an AND gate** (Foley-Omni: Synchformer≥0.2 ∧ ImageBind≥0.3) — purity first

**The physical lower bound of sync filters (new evidence, coexisting with the threshold recommendations above)**: using a controlled perturbation spectrum (global offset at five tiers of 50–500ms, local jitter at three tiers of 30–700ms, speed change at ten tiers of 0.8×–1.25×), AV-SyncBench found that **at the 50ms offset tier, the discrimination accuracy of SOTA sync models is only about 0.51 (close to random guessing)**. This means Synchformer/SparseSync-style models, when used as filters, have **an effective resolution floor of about 50ms and above** (about 1.2 frames at 24fps); mismatches below this magnitude cannot be reliably detected. This does not conflict with Ovi's earlier recommendation of "|offset|≤3 frames ∧ conf>1.5" — the two apply under different conditions: **3 frames ≈ 125ms, within the detector's reliable range, so it remains a valid engineering threshold**; but tightening the threshold to within 1 frame (≈42ms) in pursuit of extreme purity means the filter's output is already noise — tightening no longer yields any real purity gain, and instead wastes data volume for nothing — **the tightening of sync thresholds has a hard floor set by detector resolution** (AV-SyncBench). The same measurements also show that no single model handles both filtering tasks well: ImageBind is strongest on timbre/semantic tasks (0.859) but weak on timing; Synchformer/SparseSync excel at temporal offset; CAV-MAE excels at local jitter and speed changes — this provides direct model-capability-level evidence for the "temporal/semantic split" described at the start of §4, and in practice they **should be combined rather than chosen from**.

**Lip-sync sub-pipeline** (speech subset): first ASD to remove off-screen audio (Unison's lip-filtering, Vidu S1) → run SyncNet within the face bounding box (avoiding SNR dilution from full-frame analysis, Unison) → **|offset|≤3 frames ∧ conf>1.5** (proposed by Ovi, inherited verbatim by SkyReels-V4 and OmniCustom, now a de facto community standard); a high-quality subset tightens further to LSE-D≤9.5 ∧ LSE-C≥4.5 (MOVA uses this to filter out a 2.5M lip-sync subset). Ovi's experiments conclude that even a very small amount of out-of-sync data damages lip-sync capability — **err on the side of strictness**. Multi-person scenes use an all-must-pass rule + SyncNet to attribute "who is speaking" (OmniHuman).

**ASD as a two-in-one filter (the most streamlined industrial approach)**: Light-ASD enforces two exclusion rules — ① the audio is not synchronized with the active speaker on screen; ② there is no active speaker on screen at all — simultaneously accomplishing both "removing temporal desync" and "removing off-screen sound sources" in a single pass; the accompanying precondition is "retain only videos where a face is continuously visible throughout the entire sequence," on the reasoning that learning lip-sync requires the face to be visible the whole time (Wan2.2-S2V).

**An advanced use of SyncNet: upgrading from filter to structural parser** (SpeakerVid-5M). On two-person interaction data, sync detection takes on three roles beyond quality filtering: ① **audio-picture identity binding** — 3D-Speaker voiceprint diarization picks out the two main speakers (together accounting for 80%+ of total speaking time), then SyncNet confidence binds the audio-side speaker ID to the human bounding box detected by YOLO, answering "which person on screen does this voice belong to"; ② **speaking/listening state determination** — in same-frame scenes, if only one person is speaking and the SyncNet-score gap between the two exceeds a threshold, the lower-scoring party is labeled "listening"; in non-same-frame scenes, if ASR is valid with high transcription confidence but SyncNet score is low, the person is likewise labeled a listener (from which a "listening" branch is derived — a rare design that explicitly models "who is listening" as a label); ③ **cross-clip identity correction** — ArcFace facial cosine similarity verifies consistency, and outlier samples are reassigned if they are more similar to a different ID. The three steps together answer "who is speaking, who is listening, where are they on screen, and is this the same person across clips," ultimately yielding 83K (single-person branch)/16K (dialogue branch) globally traceable speaker IDs. The three sync-metric outputs (offset/confidence/embedding distance) are persisted per clip to metadata for downstream on-demand filtering, consistent with CineDance's "store all scores in metadata" approach.

**Warning**: SyncNet gives inflated scores for flat environmental noise, so audio must first be stratified by type before applying per-stratum thresholds (Script-a-Video); using the same model for both filtering and evaluation (SyncNet for filtering + Sync-C for evaluation) risks circular reasoning (ITS-JAVG).

**BGM handling**: general soundscape models do not separate audio, keeping the native mix (Ovi/MOVA/HunyuanVideo-Foley); person-centric data is the exception — Demucs/Mel-RoFormer split vocal/background tracks for separate supervision (OmniHuman, Unison).

---

## 5. Captioning Approach

**Model selection**: AV captioning requires audio-perceptive models; a purely visual VLM cannot do the job (Ovi/Foley-Omni/OmniHuman all emphasize this). Three routes to choose from by budget:

- **Open-source division of labor + large LLM fusion** (MOVA: MiMo-VL-7B for vision + Qwen3-Omni for transcription/audio + GPT-OSS-120B for fusion/adjudication — the largest model is placed at fusion rather than perception)
- **Closed-source model labels premium data, distilled into an open-source model for full-scale labeling** (Script-a-Video: Gemini-2.5-Pro labels 500K clips → fine-tunes Qwen3-Omni, with the student's error rate slightly better than the teacher's; NAVA uses a two-tier setup)
- **In-house fine-tuning** (worth it the larger the data volume: Movie Gen fine-tunes LLaMa3-Video 8B/70B; SkyReels distills a 72B teacher into Qwen2.5-VL-7B, with film/TV-field accuracy of 76.3% surpassing the teacher)

Captioning is the throughput bottleneck of the whole pipeline (NeMo Curator), requiring FP8 quantization / vLLM serving.

**Captioner ecosystem and selection patterns** (captioner ecosystem topic, synthesized across 30+ works):

- **Tier one · captioners specifically trained for video** (highest downstream reuse rate): Tarsier2-7B (adopted as base by Seedance 1.0, reused by Goku and LongCat — the single most widely adopted open-source captioner on the generation side; its natural description of camera motion is a unique advantage), ShareCaptioner-Video (its **DiffSW differential sliding window** converts "whole-frame → caption" into "detailed first-frame description + incremental step-wise differential description," naturally carrying temporal increments and supporting linear extension to arbitrary durations — the only architecture designed specifically for scalability), CogVLM2-Caption (the only reproducible step in the CogVideoX pipeline), SkyCaptioner-V1 (Qwen2.5-VL-7B base + three sub-expert fusion distillation, 76.3% accuracy on film/TV fields), AuroraCap
- **Tier two · general-purpose VLM direct labeling** (lowest cost, most common): the Qwen series is already the de facto 2025–2026 standard (Qwen2-VL/2.5-VL/3-VL); **choosing a sparse MoE architecture (~3B activated) to balance throughput and quality is a new 2026 trend** (Motif uses Qwen3-VL-30B-A3B; Allegro uses Aria, 25.3B MoE/3.9B activated, able to label 256 frames in 10 seconds)
- **Tier three · omni-modal captioners** (an explosion since 2025Q4, an AV-generation necessity): AVoCaDO (Qwen2.5-Omni-7B + GRPO, reward = five-dimension checklist coverage + dialogue F1 + length regularization, Apache-2.0), AVSCap-7B (AVSCapBench overall score 60.44, matching Gemini-3-Pro's 60.97), video-SALMONN 2/2+ (MrDPO multi-round DPO, caption error rate down 28% relative to baseline), Qwen3-Omni-Captioner / Qwen3.5-Omni (the latter explicitly positions "script-level description + automatic scene splitting + timestamp labeling + character-audio relationship description" as a capability for generating video-synthesis training data)
- **Tier four · closed-source APIs as teachers**: GPT-4V (ShareGPT4Video-40K, Koala-36M seed set, MiraData), Gemini 2.5/3 series (Script-a-Video 500K, Harmony 4M, Foley-Omni's primary annotator — chosen explicitly for "native support for joint video+audio input, since a purely visual VLM cannot judge audio existence")

Five selection patterns: ① **parameter count generally converges around 7B** (hundred-million-scale labeling requires controlling per-sample cost), with large models placed at fusion/adjudication/teacher stages rather than the perception stage; ② **multi-model division of labor beats a single model doing everything** — Panda-70M provides the strongest evidence: the single best captioner covers only 30.8% of samples, a greedily chosen set of 8 covers 76.8%, and all 31 together cover 84.7%; ③ **camera-motion recognition should always use a dedicated classifier rather than a VLM** (Movie Gen's 16 categories, SkyCaptioner's 6DoF sub-expert at 89% accuracy, Hunyuan's/LongCat's dedicated classifiers); ④ **distillation to cut cost is standard practice** (closed-source teacher → 7B student, done in nearly every work); ⑤ **the 2026 watershed is "can it hear"** — an omni captioner has become a required component for AV generation models. The only work to publish an explicit "large/small model mix ratio" figure is Movie Gen: 70% of training-set captions come from the 8B model, 30% from the 70B model.

**The caption-length spectrum and the hard constraint of the text encoder**: spanning 60× from Panda-70M's 13.2 words to UltraVideo's 824.2 words; the rough rule is that **each order-of-magnitude drop in data volume corresponds to one step up in caption length and filtering strictness** (13.2→202→318→824 words). But long captions carry two empirically demonstrated lessons: ① **the text encoder's capacity must be matched in advance** — MiraData, because its 318-word captions don't fit into CLIP's 77-token limit, switched to Flan-T5-XXL (512 tokens); LVD-2M was less fortunate — its 88.7-word captions were truncated by a frozen CLIP, and the authors directly attribute the limited improvement in I2V text matching to the 77-token cap; ② **a distributional mismatch between training captions and user prompts backfires on evaluation** — VBench prompts average only 7.6 words, inherently disadvantaging datasets built on long captions. **The encoder's token limit directly caps the usable upper bound of caption density**, so choosing a caption structure simultaneously locks in the encoder choice and the inference-side prompt rewriter (captioner ecosystem topic). UltraVideo offers the simplest antidote to this mismatch: with 1/3 probability pick one of {Brief, Detailed, Summarized}, and if either of the first two is chosen, further randomly append one of the remaining 7 structured categories as the final prompt.

**Caption style constraints (a severely underrated engineering detail)**: CogVideoX Appendix G explicitly forbids stock phrases like "The video presents/depicts/showcases" and "throughout the video," and forbids newline characters; LTX-2's principle is "comprehensive yet factual," describing only what is seen and heard, **explicitly forbidding emotional interpretation** (to prevent captions from introducing uncontrollable subjective signal into T2V training); MiraData adds the constraint of "no frame-by-frame description, no phrases like 'first frame.'" These three are the most valuable style-design lessons in the whole ecosystem.

**Caption structure**: long and dense (100–250 words) + structured slots, consistently covering: subject appearance → action → scene → camera work/shot scale → lighting → style (LTX-2, Movie Gen ~100 words, HunyuanVideo's seven-field JSON). Style principles: state only facts, no stock openers, chronological order (LTX-2, CogVideoX). **Multi-granularity is standard**: short/medium/long tiers coexist + random sampling at training time (Motif 0.5/0.3/0.2) or field dropout (HunyuanVideo), addressing the distributional mismatch between long training captions and short user prompts; the inference side pairs this with a prompt rewriter (CogVideoX/MAGI-1). Controllable tags embedded inline: camera-motion category (Movie Gen's 16 categories + 6 shot types), quality tags, "This video has no subtitles." (MOVA).

**AV joint caption schema** (a core 2026 point of divergence, three routes):

1. **Fused, single-segment full soundscape** (LTX-2: precise dialogue transcription + speaker/language/accent + full coverage of music/ambient sound; MOVA labels three tracks separately then fuses)
2. **Inline tagging** (Ovi: `<S>dialogue<E>` interleaved chronologically + a soundscape block at the end, with ablations proving unified encoding beats separate encoding; ALIVE inlines dialogue/sound effects on the same timeline)
3. **Factorized fields** (Foley-Omni's three independently verifiable, independently maskable fields `[WORDS]/[AUDIO]/[MUSIC]`; Script-a-Video's four-stream MTSS — under the same architecture, human-evaluated audio-visual alignment +56%, WER 0.84→0.13)
4. **Template-concatenated three-segment structure + component-wise random masking** (Wan 2.1 V2A): caption = dense video description + ambient sound description + background music analysis (the latter covering style/rhythm/melody/instrumentation), produced separately by a visual captioner and Qwen2-Audio, then concatenated in a fixed sentence pattern, with **explicit support for negative statements** (template example: "The video description: a horse is running. This audio contains ambient sound: the sound of clip-clop. **This audio does not contain music.**") — making "whether music is present" itself a controllable condition. The most reusable training-side companion technique is **random masking of caption components**: at training time, the ambient-sound and music caption segments are randomly omitted with a preset probability, forcing the model to build robust cross-modal association from visual cues alone, while retaining controllability when text is present
5. **Reverse corroboration from an evaluation benchmark, supporting factorization**: VABench decomposes a unified audiovisual description via an LLM into two independent streams — a visual sub-prompt and an auditory sub-prompt — for dimension-by-dimension evaluation — **a factorized schema is the precondition for being able to do dimension-by-dimension verification**; a fused single-segment caption cannot structurally support this kind of decomposition (av_benchmarks; similarly, MiraData's six independently addressable fields are what enable dimension-by-dimension image-text consistency evaluation across camera/subject/background/style)

**Formalized schema acceptance criteria** (the most operational set currently available): a high-quality omni-modal caption must simultaneously satisfy three conditions — Acoustic Completeness (covering the three classes Speech/SFX/Music), Visual Completeness (environment, characters, actions, object interactions, camera motion, OCR), and Audio-Visual Synergy (explicit binding and temporal alignment of audiovisual events) (AVSCap). AVoCaDO's reward checklist contributes one additional unique dimension: **Cross-modal Narrative Logic** (audio and visual mutually explain/complement/guide each other).

**The asymmetry of AV joint captioning**: Ovi's captioning input takes the form of "7 uniformly sampled keyframes + the complete audio track," revealing an implicit constraint — AV joint captions naturally take a **"coarse vision, fine audio"** asymmetric form, with visual description tending toward the event/semantic level rather than frame-by-frame density (Ovi).

**General anti-hallucination design** (applicable across routes): the visual track is strictly forbidden from referencing audio, and the audio track is strictly forbidden from inferring picture content (MOVA, Vidu S1); an acoustic energy gate blanks out fields (Foley-Omni −35dB); ASR ground truth fills in placeholders to prevent dialogue hallucination (OmniHuman).

**ASR and speaker attributes**: Whisper-V3 is the mainstream choice (UniTalking/SkyReels/UniVerse-1); a standard set of 7 speaker-attribute fields: age/gender/accent/pitch/prosody/emotion/speaking rate (Ovi's system); "who is speaking" binding uses ASD (TalkNet) + face tracking + ArcFace (ALIVE, OmniHuman); an omni large model + windowing approach for speaker-character binding (95.4%) far outperforms Pyannote-type dedicated tools (~63%) (CineDance). Transcriptions retain the original language, translation is forbidden (MOVA).

**Geometric annotation**: general-purpose AV generation does not need numeric camera parameters — semantic labels suffice for camera motion (Movie Gen/HunyuanVideo/SkyReels all use classification labels). This is only added for digital-human/multi-person-dialogue scenarios (face trajectories, DWPose, face-voice matrices) or Physical AI scenarios (depth/3D boxes). Details in §5.1.

### 5.1 Geometric and Structured Annotation: A Second Annotation Paradigm Beyond Captioning

Since 2025H2, a batch of datasets has emerged centered on **geometric ground truth rather than natural language** as the core annotation (SceneScribe-1M: 1 million clips/4000+ hours; SpatialVID: 2.71 million clips/7,089 hours; WildWorld: 108 million frames captured directly from an engine; Action100M: 147 million temporally localized clips), forming a second annotation route parallel to captioning.

**Annotation content**: camera intrinsics/extrinsics + temporally-consistent dense depth + 3D point tracks as a three-piece set (SceneScribe-1M uses MegaSaM for camera and depth, TAPIP3D for 3D point tracks; the paper compares DPVO/Fast3r/MonST3R/VGGT before settling on MegaSaM, reasoning that it is more robust under sparse feature points and low camera parallax). SpatialVID contributes an additional design with the highest transfer value to the generation side: **discretizing continuous camera poses into controlled cinematography terminology** (dolly in / pan left / truck right, etc.) forming "serialized motion instructions," which lets geometric ground truth be directly consumed by text models — this is the natural interface between the geometric-annotation route and the captioning route. WildWorld takes the engine-capture route, obtaining 119 columns of noise-free, per-frame ground truth (skeletal keypoints, world state, camera intrinsics/extrinsics, lossless depth, discrete action labels). Action100M is instead "structured action annotation" rather than 3D geometry: temporal localization boundaries + open-vocabulary action labels + actor identification + a hierarchical caption tree.

**The most important counter-intuitive finding (a direct warning for general pipelines)**: **conventional video generation pipelines remove samples with excessive motion, while geometric-annotation datasets instead remove samples with insufficient motion** — SpatialVID found empirically that **over 80% of Panda-70M clips fail to be successfully reconstructed by MegaSaM due to insufficient parallax/motion**, so it had to actively source high-motion material such as walk/tour/drone footage, resulting in 80% of the final library having curved or turning trajectories; SceneScribe-1M likewise uses "motion diversity" as its primary initial screening axis, with Qwen2.5-VL-72B directly discarding samples of undetermined motion intensity, substantially shrinking the source pool as a result. **Conclusion: the "anti-jitter/pursue-stability" filtering orientation of general datasets runs opposite to the "want motion" requirement of geometric annotation — the two data types cannot share the same motion-filtering chain and must fork within the funnel** (collection of geometric/structured annotation datasets).

**Geometricized motion-filtering metrics**: SpatialVID replaces scalar optical-flow magnitude with three camera-trajectory metrics — MoveDist (total displacement), RotAngle (cumulative rotation angle), and TrajTurns (number of direction changes) — plus an acceleration-based detector to identify abrupt, non-physical jitter. Compared to a scalar optical-flow score, this set of metrics can distinguish three semantically distinct motion types — "dolly push," "orbit," and "handheld jitter" — and is worth borrowing as motion-grading features in general pipelines.

**Funnel-scale reference**: SpatialVID goes from 21,789 raw hours → 7,089 hours of dynamic content (duration retention rate about 32.5%) → the HQ subset retains a further 13.7% (about 4.5% end-to-end).

**Applicability judgment**: for pure AV joint generation, this style of annotation remains optional; but if the goal includes camera-controllable generation, world models/Physical AI, or cross-shot 3D consistency, then geometric annotation is a supervisory signal that captions cannot replace — and the seven mainstream pretraining datasets listed in §9 (Panda-70M/InternVid/Koala-36M/LVD-2M/OpenVid-1M/MiraData/UltraVideo) **contain no 3D geometric annotation whatsoever**, a systemic gap in the public data ecosystem.

**Synthetic data**: the backbone must be trained on real, native audio tracks; synthetic data is used only to fill in pairings that don't exist naturally — editing pairs (InstructAV2AV), timbre-decoupled pairs (UniTalking synthesizes 6.9 million clips via TTS to prevent content leakage), TTS to backfill long-tail languages (SkyReels-V4).

**Human involvement**: "humans stay out of the annotation stream, humans stay in the decision loop": ① threshold calibration spot checks (MOVA, Open-Sora Plan's 2000-clip validation) ② seed annotation to train scoring models (CogVideoX's 20K clips, SkyReels' 93K camera-motion clips) ③ spot checks (SkyReels: pretraining 0.01%, post-training 0.1%) ④ end-of-SFT clip-by-clip curation + manual caption revision (Movie Gen, HunyuanVideo's 1M human-reviewed clips) ⑤ preference annotation.

---

## 6. Distribution Mixture and Concept Balancing

**Recommendation: lay down a three-axis domain coordinate system first before discussing mixture ratios** (the core output of the evaluation-benchmark collection topic — the taxonomies of five AV evaluation benchmarks can be directly repurposed as design references for training-data distribution, with the benefit that the training taxonomy and evaluation criteria are naturally aligned):

- **Axis 1 · content domain (using VABench's seven major categories)**: Animals / Human voice (**must distinguish linguistic from non-linguistic** — the former needs lip-sync and ASR annotation, the latter does not) / Music / Ambient sound (natural·urban·indoor) / Synchronized physical sound / Complex scenes (complex soundscapes·subjective sensation·world knowledge·symbolic association·off-screen sound sources) / Virtual worlds. One key design worth adopting alongside this: **the virtual-worlds category should be exempted from physical-plausibility quality gates**, with filtering standards set separately from realistic categories (VABench itself does not score this category on Audio/Visual Realism)
- **Axis 2 · audio type and sync difficulty (using AV-SyncBench's three-way taxonomy + ten scenarios)**: Voice / Music / Sound, subdivided into actions, animal sounds, object sounds, ambient sounds, group vocalization, single-person speech, dialogue, singing, single instrument, ensemble. This taxonomy splits by **sync difficulty rather than content topic**; its distinction between "single sound source vs. multiple sound sources" (single-person speech vs. dialogue/group vocalization, single instrument vs. ensemble) directly determines the mixture requirement for multi-source overlapping samples, and also determines domain-specific selection of sync filters (Synchformer-type models become less reliable in multi-source scenarios)
- **Axis 3 · physical-acoustics coverage checklist (using PhyAVBench's six dimensions, 41 test points)**: sound-source mechanics / fluid and aerodynamics / sound propagation environment / observer physics / time and causality / complex coupling and extreme physics. This is a checkable audit sheet rather than a mixture table, used to uncover physical blind spots in training data; empirical evaluation has already pinpointed **fluid dynamics and sound propagation environment as blind spots shared industry-wide**, which can be prioritized for supplementary data
- **Orthogonal constraint layer (using AVBench's quota rules)**: Hard Quota-Based Greedy Sampling **forces any single attribute to make up ≤50%** of the total, and stratifies difficulty as Normal (1–2 speakers) : Hard (3–4 speakers + speech overlap + noisy background) ≈ 3:1. Both of these can be directly ported as concept-balancing sampling strategies and hard-example mixture anchors
- **Real-world distribution calibration (using Omni-Judge's VidProM prompt distribution)**: the first three axes are all expert-designed idealized taxonomies, prone to drifting from real needs; Omni-Judge's 300 prompts are sampled directly from real user distributions via VidProM, and **using them to weight the aforementioned categories** avoids over-investing in long-tail categories
- **Final form**: a three-dimensional mixture matrix (content domain × audio type × physical test-point coverage), weighted by VidProM, constrained by AVBench's quota rules on single-attribute share, audited for blind spots via PhyAVBench's checklist, with filter domain selection driven by AV-SyncBench's scenario tags. **Known gaps**: multi-shot long-form narrative, multilingual accents, and cross-shot audio-track continuity currently have no benchmark to validate against, so investment in these domains cannot yet be verified against public benchmarks

**Concept balancing = embedding clustering + inverse-frequency resampling** (strong consensus): VideoCLIP clustering into 10K concept centers (HunyuanVideo), 120K+ clusters doubling as outlier quality checks (Step-Video), sampling by 1/√cluster-size (Movie Gen). The cost of balancing can be as high as halving the data (SkyReels-V2).

**Deliberate up-weighting of people**: the HD set requires ≥60% of clips to contain people + a targeted-retrieval taxonomy of 600 human-action verbs (Movie Gen); 9 major categories/86 subcategories explicitly balanced with a tilt toward people (Goku). Negative list: games/animation/screen recordings/talking-head video (Cosmos, CogVideoX, SkyReels).

**Audio-category mixture** (the biggest point of divergence): speech-dominant (MOVA 69.47%, Foley-Omni 66%) vs. sound-effect-dominant (Movie Gen sound-effects:other ≈10:1, UniVerse-1 non-speech 84.6%). For building a general-purpose model from scratch, the recommendation is **retain all categories + tag-based mixture control** (HunyuanVideo-Foley), with BGM managed proportionally rather than removed wholesale (ALIVE); a 2026 trend is to break out singing as an explicit fourth category (SkyReels-V4, Apollo). Staged scheduling: "train large-scale speech first → mix in sound effects to keep speech alive" (Ovi; NAVA flips its audio:AV ratio from 3:1 to 1:2).

**Language**: Chinese-English bilingual is the industrial standard (MOVA: Chinese TV dramas + English YouTube; Seedance 2.0 covers dialects/opera; Kling 3.0 covers five languages + accent control); accent as an explicit annotation field (LTX-2, Ovi).

**Aligning mixture with evaluation benchmarks**: training taxonomy corresponds one-to-one with evaluation categories (Movie Gen's five concept categories; Cosmos's five data domains ↔ PAI-Bench's seven domains); a closed loop of evaluation weakness → dynamically supplementing corresponding-category data (MAGI-1, Apollo, Seedance 1.0). Note that Data-Juicer's approach of "embedding VBench metrics directly into recipe search" is the most thorough but carries a risk of overfitting to the benchmark.

---

## 7. Training Curriculum and Data Scheduling

**Visual-backbone pyramid curriculum** (consensus):

1. **T2I foundation**: pure images establish visual priors (Movie Gen proves that joint training from scratch is significantly slower; LongCat-Video's T2I stage makes up 42% of total iterations), images continue to be mixed in throughout all subsequent stages to prevent visual-quality degradation (HunyuanVideo, Goku, MAGI-1 keep a constant 4:1 image:video ratio)
2. **Progressive resolution + shrinking data volume**: CogVideoX 256→480→768px, batch 2000→100; Open-Sora 2.0 three stages 70M@256px→10M→5M@768px (the high-resolution stage switches to I2V to save compute)
3. **Late stages raise frame rate only, not resolution** (Seedance 1.0, HunyuanVideo 1.5: 16→24fps)
4. **Annealing**: brief fine-tuning on the top 10–20% highest-quality subset (CogVideoX stage 4 uses top 20%/10K steps, at the cost of slight semantic-capability regression); 4K cooldown (Cosmos, 388K clips)

Stage-switching is judged by "convergence + saturated visual quality" rather than fixed step counts (Cosmos); a sharp loss drop upon switching to higher-quality data can serve as a switch signal (Step-Video). **Quality stratification as curriculum**: the same corpus is nested into progressively tighter subsets by threshold, one per stage (Allegro's aesthetic score 4.8→5.3, with expensive metrics only computed after the data has already shrunk by two orders of magnitude — a key cost-design choice).

**Audio-video modal curriculum** (de facto standard, adopted across Ovi/MOVA/Harmony/ALIVE/SkyReels-V4/Unison):

- **Massive pure-audio pretraining of the audio tower first → a small amount of paired data for joint adaptation** (the audio stage runs far more epochs than the joint stage; JavisDiT++'s ratio is 50:2)
- **Ablation evidence**: skipping this curriculum and jointly training directly causes audio collapse — Foley-Omni WER worsens 4×, InstructAV2AV FAD worsens 88% (audio gradients get drowned out by visual gradients)
- **Protective mechanisms during the joint stage**: loss weighting λ_v=0.85/λ_a=0.15 (Ovi); asymmetric learning rates, video 1e-5/audio 1e-6 (ALIVE); mixing in 50% pure video to prevent visual degradation (SkyReels-V4); replay ratio to prevent forgetting as high as 70% (Foley-Omni)
- **End-to-end beats training a frozen-tower bridge**: MOVA found empirically that "freeze both towers, train only the bridge" plateaus early; switching to three-tower end-to-end training + text dropout from 0.5→0.2 forces cross-modal alignment, with LSE improving monotonically stage by stage
- **A strong base model can skip the curriculum entirely**: UniVerse-1 (Wan + Ace-step dual experts) uses a single stage + LQLS instead of data-quality tiering

**Dynamic mixture**: adaptively adjusting the sampling ratio based on evaluation-metric feedback (MAGI-1, Apollo, Seedance) is displacing static curriculum tables; an alternative approach is "independent per-domain SFT + model-soup parameter merging" to sidestep mixture-ratio tuning (Cosmos).

---

## 8. Post-Training: SFT and Preference Alignment

**SFT**: scale is roughly 0.4–1.5% of pretraining volume (HunyuanVideo's 1M manually curated clips, using four aesthetic criteria + three motion criteria; NAVA's retention rate is 1.07%, with a strong model used to rewrite captions — double purification of both data and annotation); concept-balanced k-NN + manual selection for cinematic feel (Movie Gen, using only 512 GPUs); high-aesthetics:realistic ratio 3:1, training only 0.5 epoch to prevent overfitting (ALIVE). Quantified gains: Movie Gen's video SFT yields an overall quality net win rate of +34.65; audio fine-tuning uses <1.1% of pretraining volume for gains of +24.9~+43.0 across various dimensions.

**Preference alignment**: the industrial standard is a four-stage pipeline of "pretraining → CT → SFT → RLHF" (Seedance, Kling 3.0, HunyuanVideo 1.5). The approach has evolved as follows:

- Offline DPO: 30K preference pairs to train a Bradley-Terry RM + three rounds of 20K pairs each (SkyReels-V2); GSB human annotation (HunyuanVideo 1.5)
- **2026 trend: online RL with GRPO + multiple reward models** (LongCat-Video, Cosmos: HPSv3 + VideoAlign, with graded input isolation for motion evaluation to suppress reward hacking)
- Audio-video-specific: AV-DPO with 25K preference pairs, six-model multi-dimensional reward, "modality-aware ranking" to avoid contaminating the signal with high-quality-audio-paired-with-low-quality-video (JavisDiT++); audio-video-customized RLHF + multi-dimensional RM (Seedance 1.5 pro)
- RM for lip-sync/timbre dimensions remains a current gap, and open-source work is almost entirely missing preference alignment — this is the largest gap relative to closed-source work

**Distillation**: low-cost last-mile distillation is now standard (<1K steps; Vidu S1's three stages: bidirectional teacher → causal adaptation → DMD distillation).

---

## 9. Why Do It This Way: The Strongest Quantitative Evidence

| Conclusion | Evidence | Source |
|------|------|------|
| Data processing quality is the biggest lever | Same source material, same architecture; halving data volume yields VBench +9.67 points (semantic +28.2) | Koala-36M vs. Panda-70M |
| Quality over quantity | A 12.09% retention rate tops VBench (82.53%) at 22× less compute; high-quality data ×6 repeats > lower-quality data ×8 compute | Data-Juicer Sandbox |
| Structured captions improve semantics/dynamics | Swapping only the caption: text alignment 7.73→15.36, dynamics degree +107% | MiraData |
| Structured AV scripts yield zero-cost gains | Same architecture, only the annotation format changes: audio-visual alignment +56%, WER down 85% | Script-a-Video |
| High-quality SFT is worth the manual investment | <1% premium rate, overall quality net win rate +34.65 | Movie Gen |
| Native HD > low-res + super-resolution | VQAA 73.46 vs. 69.26, the one dimension that cannot be compensated for by filtering | UltraVideo/OpenVid |
| Isolating low-quality data beats discarding it | LQLS (loss applied only at t>800): removing it drops ID consistency 0.89→0.78 | UniVerse-1 |
| The audio-tower curriculum cannot be skipped | Direct joint training: WER×4 (Foley-Omni), FAD +88% (InstructAV2AV) | Foley-Omni / InstructAV2AV |
| Small model + precise data can leapfrog | 2B parameters, <10M clips, VBench 83.76%, surpassing Wan2.1-14B | Motif-Video 2B |
| Low cost can approach SOTA | $199.6k full training run, 0.69% gap from Sora | Open-Sora 2.0 |

(Weaker evidence to cite cautiously: HunyuanVideo 1.5's "0.125% retention rate + 8.3B outperforming larger models" has no controlled comparison; Ovi's "even a small amount of out-of-sync data damages lip-sync" experiment was not disclosed in detail.)

---

## 10. Data Infrastructure and Cost Reference

- **GPU route**: NeMo Curator is the industrial de facto standard — 2000×H100 processes about 1M hours of 720p video in one day, an 89× speedup over CPU at equal power draw, sharding directly by "resolution × aspect ratio × duration" buckets to feed straight into curriculum training
- **CPU route**: Data-Juicer — operator reordering saves 70% of time, 8 nodes deduplicate 5TB in 2.8h
- **In-house**: Seedance (BMF+Ray three-tier heterogeneous), MOVA (Ray + GPU/NPU hybrid)
- **Throughput as training payoff**: an offline bucket-balanced sampler raises GPU utilization from 20%→90% and training throughput 5.4× (Motif-Video)
- **Cost anchors**: Open-Sora 2.0 full run $199.6k (11B, excluding data processing); MOVA ~43,000 GPU-days; Movie Gen peak 6144×H100
- Common lesson: captioning is the recognized bottleneck, place it last and serve it; I/O rather than compute is often the actual bottleneck

---

## 11. Seven Trends for 2026

1. **Multi-shot narrative samples entering training**: the single-shot consensus is being broken (Seedance 2.0 clips allow multiple coherent shots, Kling 3.0's six-shot Director Memory, CineDance-1M averaging 24.2 shots/92.8s)
2. **VLM/LLM semantic quality inspection replacing shallow scorers**: final reviewer (Cosmos/Vidu S1), unified scoring+labeling (MAGI-1), large-model cross-checking (MOVA uses a 120B model for adjudication + hallucination self-audit); but a single scorer can be gamed, requiring a combination of multiple verifiers (ITS-JAVG)
3. **Structured/temporally-grounded captions becoming the competitive focus**: factorized streams, entity-level anchor tokens bound across shots (Script-a-Video, CineDance); a structured schema itself provides zero-cost improvement
4. **Evaluation-feedback closed-loop dynamic data scheduling** replacing static curriculum tables (MAGI-1/Apollo/Seedance)
5. **Online RL with GRPO + multi-dimensional reward** replacing offline DPO; AV-specific RM (lip-sync/timbre) remains a gap to fill
6. **"Train short, infer long"**: ≤20s training + chunked concatenation to generate 5-minute output (StreamChar, LongCat continuation)
7. **The rise of small-data-high-efficiency routes**: strong base model + narrow-domain precise data competing against big data (CCL 4M vs. Ovi 30.7M, Motif-Video 2B, UniAVGen 1.3M)

---

*This document is synthesized from the wiki's 7 horizontal comparison pages. Full figures and sources for each item are in the corresponding comparison pages: [Data Scale and Distribution](topics/data-scale-distribution.md) · [Cleaning Pipeline](topics/cleaning-pipeline.md) · [Captioning/Annotation Method](topics/captioning-annotation.md) · [Audio-Video Alignment](topics/av-alignment.md) · [Training Coordination](topics/training-strategy.md) · [Effectiveness Comparison](topics/data-ablation-evidence.md)*
</content>
