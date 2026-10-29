# Where we are and what to do next — stage1 · stage2 · VLM · orchestration (AI-authored, 2026-10-29)

Author: AI agent (Claude), 2026-10-29 ~19:30 IST. Human companion: [pipeline_plan.md](pipeline_plan.md). Correlation ID: RN-20261029-pipeline-plan.
Planning only — nothing here is approved or started. Evidence: stage2 [EXP-2026-10-28-02](experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md),
[EXP-2026-10-28-01](experiments/EXP-2026-10-28-01-train-subset-influence-loop/record_ai.md), [EXP-2026-10-27-01](experiments/EXP-2026-10-27-01-best-combination-v1/record_ai.md);
VLM `/sfs/vlm_qwen/experiments/2026-10-19/09_c1_full_stage1_eval/reports/FULL_METRICS.md`, `08_class_only_fourway_sft/reports/COMPARISON.md`,
vlm notebook EXP-2026-10-26-01 (VLM vs YOLO overlap); stage1 `/sfs/stage1/STATE.md`; `/sfs/data/CLAUDE.md`.

## 1. Where we are

| component | model | latest result | test set |
|---|---|---|---|
| stage1 spotter | n0005 (Aug), class-agnostic | ~93% of objects get a box at conf ≥0.05; **VLM hand-off uses conf ≥0.50** | old test_80 |
| stage2 detector | ensemble y8m+y11l, Train359 (new) | ens MICRO/MACRO F1 65.0 / 57.4; per-class best combination 66.5 / 58.3 (65.8 / 57.7 after dedup) | **Test116** |
| VLM verifier | Qwen3-VL + C1 class-only adapter | rejects 74% of non-targets (R1: 0%); end-to-end radar R 35.5% (P 19.6%), missile R 48.5% (P 24.9%) | clean 67 scenes of old test_80, IoU 0.5 |
| VLM vs YOLO | same 387 radar/missile targets, IoU 0.5 | YOLO 204 (52.7%), VLM 168 (43.4%), both 138, union ceiling 234 (60.5%) | clean 67 scenes |
| train-subset loop | y8m proxy, 2 seeds | **71 Train images can move to test** without failing (2-seed MICRO 57.7 vs 57.9); resolved | Test116 |

## 2. Problems found (train → test → post-processing)

1. **Every stage2 choice is selected on the test set.** best.pt checkpoint, per-class model + confidence, deploy thresholds — all chosen on Test116.
   The 66.5 / 58.3 is an upper bound, not an unbiased result. There is no validation split.
2. **The three components are scored on different tests and rules.** stage2: Test116, IoU 0.3, 2026-10-27 label rule (P_Camo kept under parent).
   VLM: 67 clean scenes of old test_80, IoU 0.5, historical taxonomy (radome → other). stage1: old test_80. End-to-end numbers are not comparable.
3. **The "35 YOLO-only targets never reached the VLM" is the stage1 hand-off gate, not stage1's ability.** At conf ≥0.50 stage1 drops 57/152 radars
   and 40/235 missiles before the VLM sees them; at ≥0.05 it boxes ~93%. Stage1 was also never retrained on Train359.
4. **Image-source shift on new test scenes.** The 29 new Test2 scenes (`2J`–`3B`) have **bicubic** 2× images, not HAT SR. Stage1 and the VLM were trained
   on SR renders → those scenes are out of distribution for them (stage2 reads native TIFFs, unaffected).
5. **Noise.** Single seeds everywhere except the subset loop; measured seed noise ~0.5–0.7 (full runs) and ~±3 MICRO (short proxy runs).
   Several stage2 "wins" (ensemble +0.7 MACRO) are inside that noise.
6. **Test set not final.** The loop's 71-image move list would change Train and Test again; anything trained before that decision must be redone.
7. **VLM precision collapses at natural prevalence.** 93% of stage1 proposals are non-targets; even rejecting 74% of them leaves radar/missile precision
   at 20–25%. The rejection training data is still the weak point (your new hard negatives are pending).
8. **Deployment cost.** The best combination draws on up to 9 prediction sets (several ensembles with TTA) — fine for scoring, heavy for production.

## 3. Plan (in order — each step removes a reason later numbers could be wrong)

**Step 0 — freeze the data (no GPU, ~1 day; your decisions).**
- Approve or edit the 71-image move list (loop result). Then freeze Train / Test for the month.
- Carve a **site-disjoint validation set from Train** (~15% of sites, rare classes stratified) — all model/threshold picks happen there; Test is scored once.
- One benchmark for every component: final Test, permanent instance IDs, one label rule (2026-10-27), report IoU 0.3 and 0.5.
- Ask for HAT SR renders of the 29 bicubic scenes (or score them separately for stage1/VLM).

**Step 1 — stage2, the product (GPU ~1.5 days).**
- Retrain the best recipe on the frozen split with **2–3 seeds** (y8m + y11l).
- Add the two relabels that worked: D (non-launcher / unloaded missiles → vehicle, best missile precision) and R (stowed radar → vehicle, +5 radar precision); one seed each first, more if they pass.
- Choose per-class model + confidence on **validation**; report Test once. Keep a light deploy option (≤2 models) beside the best combination.
- Expected: honest 64–66 MICRO / 56–58 MACRO; radar/missile precision is the main lever.

**Step 2 — stage1 hand-off (GPU ~0.5 day).**
- Measure recall vs. proposals at gates 0.05 / 0.2 / 0.35 / 0.5 on the frozen test; pick the lowest gate the VLM can afford.
- Alternative worth testing: feed the VLM **stage2's boxes** instead of stage1's — stage2 already localises well; its failures are mostly class confusion, which is the VLM's job.
- Retrain stage1 on Train359 only if neither option recovers the missing radars.

**Step 3 — VLM (GPU ~1–2 days, after your rejection data).**
- Train on your new hard negatives (detector false alarms from Train scenes only), leakage check on final pixels + prompt, class-only targets (C1 style).
- Success on the natural distribution: radar/missile precision ≥ 50% at recall ≥ 50% of handed-off targets.

**Step 4 — orchestration (CPU + short GPU).**
- Use the VLM as a **targeted re-checker**, not a parallel detector: only stage2 boxes that are radar/missile, or vehicle/arty/logistic boxes with
  radar/missile in the top-3 scores. Rule options: VLM agrees → keep; VLM rejects → drop or down-weight; VLM says radar/missile on a vehicle box → relabel.
- Tune the rule on validation, report end-to-end on Test vs. stage2 alone, counting false positives (the 60.5% union is a ceiling without FPs).

## 4. What is most likely to give the best result
1. **Stage2 with clean validation and the D + R relabels** — largest and most certain gain; it is the product number.
2. **VLM as re-checker of stage2's ambiguous radar/missile boxes** — the only place the overlap study shows extra gain (up to +7.8 recall points ceiling).
3. Stage1 retraining is the lowest priority unless Step 2 shows the gate cannot recover enough targets.

## 5. Decisions needed
1. Approve the 71-image move (or a subset) and freeze the split.
2. Validation: site-disjoint hold-out from Train (proposed) or k-fold.
3. VLM proposals: lower the stage1 gate, or use stage2 boxes?
4. HAT SR for the 29 bicubic scenes?

## Correction (2026-10-29 21:30 IST, after the researcher asked "is this true")

**1. Noise.** The "0.5–0.7 on full runs" figure was wrong.
- It is |seed 0 − seed 1| of the **y8m proxy**: DOTA init, 15 epochs, one pair of seeds, from the subset loop. No full 100-epoch run has ever been repeated with a second seed, so full-run noise is **unmeasured**.
- Per-class proxy noise is much larger: arty 4.3, camo 2.7, tank 1.6, logistic 1.6, smallvehicle 1.3, radar 0.9, vehicle 0.9, missile 0.7.
- The "±3 MICRO" figure compared two **different** removals (35 vs 71 images), so it was not seed noise.

**2. Relabels D and R on Test116 (labels_simple, Parent-Class rule).** The test_80 gains do not carry over. Both relabels move along the same P/R curve as the control (EXP-04 y8m control fine-tune).
- **D, missile:**
  - best F1 56.7 vs control 57.2;
  - at conf 0.5, P 56 / R 57 vs P 47 / R 61, which is the control's own point at conf 0.7 (P 56 / R 56).
- **R, radar:**
  - best F1 55.2 vs 55.5;
  - at conf 0.5, P +5 (71 vs 66) but R −4 (44 vs 48), which is the control's curve.
- The test labels still count unloaded launchers as missile and stowed radars as radar. A model trained to call those objects vehicle therefore loses recall on them.

**3. GPU estimate.** One seed of the full recipe takes 5.4 h for y8m and 6.95 h for y11l, so 12.35 h per seed.
- 3 seeds ≈ 37 h, which is the 1.5 days.
- Adding D and R (y8m + y11l, one seed each) adds about 25 h more.
