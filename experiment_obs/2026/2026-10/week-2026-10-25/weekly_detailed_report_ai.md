# Week 25–31 Oct 2026 — detailed report: stage 2 detector + VLM verifier (AI-authored)

Author: AI agent (Claude), 2026-10-29. Human companion: [weekly_detailed_report.md](weekly_detailed_report.md). Correlation ID: RN-20261029-weekly-detailed-doc.
Sources are linked in each section; the new error analysis is `artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/ERROR_CAUSES.md`
(script `code/error_causes.py`, CPU only, re-run any time).

**How to read the numbers.** Unless stated otherwise, stage 2 numbers are on **Test116** (the new 27 Oct dataset: Test1 44 + Test2 72 images,
6,910 objects), with the official `labels_simple` classes and a box counted correct at **IoU ≥ 0.3**.
P = precision (share of boxes that are right), R = recall (share of objects found), F1 = their balance.
MICRO = over all objects; MACRO = average of the 8 classes (so rare classes count as much as vehicles).
VLM numbers are on the older benchmark (67 clean scenes of the old test set, IoU ≥ 0.5) — see §4.

---

## 1. Summary

- **Stage 2 is the product and it improved this week.** Retrained on the new, larger training set (Train 359) and deployed as a
  per-class mix of the best models, it reaches **MICRO F1 65.0 (P 67.2 / R 63.0), MACRO 57.3** with radar recall **63.6%**.
- **The biggest single error is missile ↔ vehicle confusion.** 88 of 181 missed missiles were called *vehicle*, and 129 of 207 wrong missile
  boxes sit on *vehicles*. It is concentrated in MLRS, launchers whose load is unknown/empty, and non-launcher missile vehicles.
- **Second: rare sub-types the model has barely learned** — K9 0/16, HQ6 0/10, self-propelled artillery 22%, EWS radar 21%, MLRS 24%.
- **Third: small, faint objects** — smallvehicle, vehicle and camo have the most "no box at all" misses and the most boxes on unlabeled ground.
- **The VLM complements YOLO but cannot be used alone.** It catches 30 radar/missile targets YOLO gets wrong, but at the real mix of
  proposals (93% non-targets) its precision is only 20–25%. Its best role is a **re-checker of stage 2's doubtful boxes**.
- **Data:** 71 training images can move to test without losing accuracy (confirmed on 2 seeds), which would make the test set bigger.

---

## 2. What was done this week

### Stage 2 (`/sfs/stage2`)

| date      | work                                                                                                                                                                                    | result                                                                                                                                                                                                                                                                   |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 26 Oct    | Where to count **camouflaged missiles and radars** (under camo, or under their own class) — [EXP-2026-10-26-01](experiments/EXP-2026-10-26-01-pcamo-missile-radar-mapping/record_ai.md) | Under their own class is clearly better: missile F1 36 → 59–62, radar 50 → 55; camo row also improves for missiles. Became the 27 Oct label rule.                                                                                                                        |
| 26 Oct    | Guide to the best prediction files for every test image — [EXP-2026-10-26-02](experiments/EXP-2026-10-26-02-best-pred-files-guide/record_ai.md)                                         | One folder per image with best boxes, GT and per-image TP/FP/FN, for visual review.                                                                                                                                                                                      |
| 27 Oct    | **Combination tool** `combine_best.py` — [EXP-2026-10-27-01](experiments/EXP-2026-10-27-01-best-combination-v1/record_ai.md)                                                            | Picks the best model + confidence per class from all experiments; reproducible after every new run.                                                                                                                                                                      |
| 27 Oct    | New dataset **27_10** checked (MLRS per scene, label policy: camouflaged radar/missile under parent class)                                                                              | Train 359 / Test116; 45 new scenes placed.                                                                                                                                                                                                                               |
| 28–29 Oct | **Smallest training subset** loop — [EXP-2026-10-28-01](experiments/EXP-2026-10-28-01-train-subset-influence-loop/record_ai.md)                                                         | 22 training runs guided by influence scores (which images help the test set). **71 of 359 images can move to test** with no measurable loss (2-seed mean MICRO 57.7 vs 57.9, MACRO 46.2 vs 45.8). Every larger removal (79–214) failed; random removal of 71 failed too. |
| 28–29 Oct | **Best recipe on the new split** — [EXP-2026-10-28-02](experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md)                                                                     | y8m + y11l retrained on Train 359 + GE booster. MACRO up on every model pairing (ensemble +0.7, y8m +1.7, y11l +1.5).                                                                                                                                                    |
| 29 Oct    | All 25 earlier prediction sets re-run on Test116; best P / R / F1 per class × confidence                                                                                                | Table in `knowledge/best_combination_ai_by_conf.md`.                                                                                                                                                                                                                     |
| 29 Oct    | **Deployment selection approved** (recall-leaning)                                                                                                                                      | MICRO 65.0 / MACRO 57.3 after merge; spec `reports/deploy_2026-10-29.json`.                                                                                                                                                                                              |
| 29 Oct    | **Error-cause analysis** of the deployed output (this report, §3)                                                                                                                       | Main failure patterns measured per class and sub-type.                                                                                                                                                                                                                   |

### VLM verifier (`/sfs/vlm_qwen`)

| date | work | result |
|---|---|---|
| 26 Oct | All-80 evaluation verified (6,983 proposals) and comparability audit vs stage 2 | The two projects' headline numbers use different tests, classes and IoU, so they cannot be ranked directly. |
| 26 Oct | **VLM vs YOLO on the same targets** — vlm EXP-2026-10-26-01 | 387 radar/missile targets: YOLO 204, VLM 168, both 138; VLM-only 30, YOLO-only 66; perfect union 234 (60.5% vs 52.7%). |
| 28 Oct | **Per-instance ledger** of every disagreement | Of YOLO-only successes, 27 never reached the VLM (stage 1 gave no box) and 39 reached it but were misclassified (mostly answered "other"). |
| 27 Oct | Own items: analyst asked about the 15 overlap images (8 found); more MLRS instances requested; end-to-end metrics for low-res prepared; final test set queried | — |

---

## 3. Main causes of the remaining errors (stage 2, deployed output)

### 3.1 Per class: where the misses and wrong boxes come from

| class        |   F1 | found / objects | misses: no box | misses: wrong class (top)                 | wrong boxes: on background | wrong boxes: on other objects (top)               |
| ------------ | ---: | --------------- | -------------: | ----------------------------------------- | -------------------------: | ------------------------------------------------- |
| vehicle      | 72.6 | 2170 / 3098     |            573 | missile 128, smallvehicle 66, logistic 63 |                        342 | smallvehicle 97, missile 91, tank 61, logistic 61 |
| tank         | 75.5 | 454 / 645       |             50 | vehicle 60, camo 39, arty 39              |                         36 | camo 22, vehicle 21, arty 18                      |
| smallvehicle | 61.0 | 741 / 1242      |            388 | vehicle 96                                |                        374 | vehicle 69                                        |
| radar        | 57.2 | 175 / 275       |             54 | vehicle 24, missile 12                    |                         72 | vehicle 28, missile 27, camo 14                   |
| camo         | 55.7 | 288 / 558       |            216 | tank 22, radar 15                         |                        117 | tank 39, vehicle 18                               |
| missile      | 54.5 | 232 / 413       |             53 | **vehicle 88**, radar 27                  |                         44 | **vehicle 129**, radar 13                         |
| arty         | 51.9 | 128 / 249       |             78 | tank 18, radar 10                         |                         26 | tank 39, vehicle 32                               |
| logistic     | 30.3 | 81 / 298        |            142 | vehicle 63                                |                         72 | vehicle 63, smallvehicle 14                       |

"No box" = no box of any class on the object. "Background" = the box touches no labeled object (empty ground, or an object nobody labeled).

### 3.2 Cause 1 — missile and vehicle look alike to the model (largest error)

- Missile ↔ vehicle accounts for **217 of 388 missile errors** (88 missed as vehicle + 129 vehicles called missile), and 128 vehicles are lost to missile.
- Recall by missile sub-type and state (labels_detailed):

| group | found / objects | recall |
|---|---|--:|
| loaded launchers (Is_Empty = No) | 200 / 293 | 68% |
| empty launchers (Is_Empty = Yes) | 22 / 41 | 54% |
| **load not recorded (Is_Empty = Nil)** | 10 / 79 | **13%** |
| **MLRS** | 20 / 82 | **24%** |
| HQ6 | 0 / 10 | 0% |
| non-launcher missile vehicles (Functionality Nil / Transloader / Dump) | 12 / 62 | 19% |
| S300 · HQ22 · HQ2 | 89 / 109 | 82% |

- **Reading:** the model recognises classic loaded SAM launchers well. It fails where the object really looks like a truck — MLRS,
  transloaders, launchers without visible canisters. This is partly a *definition* question (should these count as missile?) and partly
  too few examples.
- Earlier this month, relabelling those objects to vehicle in training (variants D and R) improved test_80, but on Test116 it only moved
  along the same precision/recall trade-off — no net gain. So the fix is not a simple relabel.

### 3.3 Cause 2 — rare sub-types are almost never found

| sub-type | found / objects |
|---|---|
| K9 (arty) | 0 / 16 |
| HQ6 (missile) | 0 / 10 |
| camo-covered transport vehicle | 2 / 30 |
| MLRS under camo | 0 / 7 |
| self-propelled artillery (SPA tracked / wheeled) | 12 / 55 (22%) |
| AA gun | 1 / 10 |
| EWS (radar) | 6 / 28 (21%) |
| ABLV (tank) | 2 / 10 |
| engineering vehicle | 73 / 168 (43%) |

- Common sub-types of the same classes are fine: PCL161/181 arty 93–96%, YLC2V / HT233 / JY27A / Cheese Board radars 84–100%, Tank 86%.
- Arty's low recall on large objects (≥ 60 m²: 6 / 59) is probably the same effect: SPAs are large and are merged into arty (not yet cross-checked per object).
- The class-level F1 hides this: a class can look acceptable while whole sub-types are invisible.

### 3.4 Cause 3 — small and faint objects: misses and boxes on unlabeled ground

- smallvehicle: 388 objects with no box and 374 boxes on unlabeled ground; vehicle: 573 and 342; camo: 216 and 117.
- Vehicles under 20 m² are found only 44% of the time (vs 75% for 20–60 m²).
- Some "background" boxes are probably real objects that were never labeled (dense car parks, convoys). This has not been checked yet;
  a sample review would tell how much of this error is the model and how much is the labels.

### 3.5 Cause 4 — radar specifics

- 54 of 100 missed radars have no box at all; 72 of 162 wrong radar boxes are on empty ground.
- Recall drops for stowed / unknown-position radars (Position Open 72%, Closed 53%, Nil 43%) and for smaller radars (20–60 m²: 39% vs ≥ 60 m²: 71%).
- EWS is the weakest radar type (21%). Camouflaged radars are found as often as uncovered ones (65% vs 63%) under the new label rule.

### 3.6 Cause 5 — logistic and vehicle overlap

- Logistic is the weakest class (F1 30.3): 142 objects get no box, 63 are called vehicle, and 63 vehicles are called logistic.
- Large logistic objects are almost never found (1 / 35). The visual line between "logistic" and "vehicle" is probably not sharp in the labels either.

### 3.7 Cause 6 — Test2 is harder than Test1 for some classes

| class | Test1 P / R | Test2 P / R |
|---|---|---|
| missile | 62 / 67 | 42 / 44 |
| tank | 87 / 87 | 78 / 61 |
| arty | 53 / 15 | 52 / 62 |

- Missile and tank are much weaker on Test2 (newer / different sites). Arty is the opposite (Test1 recall 15% on 54 objects); which sub-types sit in Test1 is not yet checked.

### 3.8 VLM-side causes

- **Proposal mix.** 93% of the boxes stage 1 hands to the VLM are not radar/missile (3,941 of 4,231). The VLM rejects 86% of them,
  but the remaining 14% still outnumber the real targets, so precision is 20% (radar) / 25% (missile).
  *(On the balanced 256-sample check it rejected 74%; the earlier R1 model rejected none.)*
- **Hand-off gate.** Stage 1 passes only boxes with confidence ≥ 0.50: 57 of 152 radars and 40 of 235 missiles never reach the VLM.
  This — not stage 1's ability — explains why 27 YOLO-correct targets were never seen by the VLM.
- **Misclassified targets mostly become "other".** Of 39 targets the VLM saw but got wrong, 27 were answered "other".
- **Image source.** 29 new Test2 scenes have bicubic 2× images instead of HAT super-resolution, which stage 1 and the VLM were trained on.

---

## 4. Where we are

| component | status | best result | benchmark |
|---|---|---|---|
| stage 2 detector | retrained on Train 359; deployment selection approved | MICRO 65.0 / MACRO 57.3; radar R 63.6, missile R 56.2 | Test116, IoU 0.3 |
| VLM verifier (Qwen3-VL-8B + C1 class-only) | trained, evaluated end-to-end | radar R 35.5 / P 19.6; missile R 48.5 / P 24.9 | 67 clean old-test scenes, IoU 0.5 |
| VLM + YOLO together | analysed, no fusion rule yet | perfect-union ceiling 60.5% target recall vs YOLO 52.7% | same 67 scenes |
| stage 1 spotter | unchanged since August | — | old test set |
| dataset | 27_10 in use; 71-image move found | — | — |

- Seed-to-seed variation of the full recipe is **not measured yet** (one seed per model). Proxy runs vary about 0.5–0.7 MICRO, and
  up to ~4 F1 for arty. Gains below ~1 point should be read as "no change" until repeated.
- Deployment cost: the approved preset needs 8 weight files and 10 model passes per image.

---

## 5. Where we are moving — and feasible next steps

Ordered by expected gain per effort. Effort uses the measured speed on the L40S: one full stage 2 seed (y8m + y11l) ≈ 12.4 h.

### Stage 2

| # | step | targets cause | effort | expected outcome |
|---|---|---|---|---|
| S1 | **Count training examples per sub-type** (K9, HQ6, SPA, EWS, MLRS, ABLV, transloaders) and list which have < 30 | 2 | CPU, ~1 h | Tells which sub-types need more data vs better sampling |
| S2 | **Review a sample of "background" boxes and "no box" misses** (e.g. 50 smallvehicle/vehicle FPs, 30 radar FPs) on the images | 3, 4 | ~2 h of viewing | Splits label gaps from model errors; label fixes are free gains |
| S3 | **Decide the scoring definition for missile-looking trucks** (MLRS, Is_Empty Nil, non-launcher) — e.g. accept *vehicle* for them, as P03 did | 1 | CPU, minutes per variant | Up to 88 missile misses + 129 FPs are in this group; a clear rule makes the metric reflect what operators need |
| S4 | **Rare sub-type oversampling** in training (repeat tiles with rare sub-types, copy-paste of rare objects) | 2 | 1 seed ≈ 12 h | Recall on K9/SPA/EWS/HQ6/MLRS; low risk to common classes |
| S5 | **Second seed of the best recipe** | — | 12.4 h | Measures real noise; needed before trusting any gain below ~1 pt |
| S6 | **Apply the 71-image move** (if approved) and retrain once | — | 12.4 h | Bigger, steadier test set; same accuracy expected |
| S7 | **Light deploy option** (≤ 2 models, single pass) scored beside the full preset | cost | CPU + 1 inference pass | Know what accuracy the 10-pass preset buys |
| S8 | Grad-CAM-style maps on camouflaged radar/missile and on missile↔vehicle errors | 1 | ~0.5 day | Shows what the model looks at (your weekly-note idea) |

### VLM

| # | step | targets cause | effort | expected outcome |
|---|---|---|---|---|
| V1 | **Move the VLM to Test116** with the same IDs, classes and IoU as stage 2 | benchmark | CPU + 1 inference pass | Direct comparison and end-to-end numbers |
| V2 | **Feed the VLM stage 2's boxes** (or stage 1 at a lower gate, e.g. 0.2) instead of stage 1 ≥ 0.50 | hand-off | 1 inference pass per option | Recovers the 57 radars / 40 missiles stage 1 drops; stage 2 boxes likely carry fewer non-targets (to measure) |
| V3 | **Hard-negative training data**: the stage 2 errors in §3 (vehicles called missile, background radar boxes) taken from **Train** scenes only, plus your supplied examples; leakage check on pixels and prompt | proposal mix | data prep ~1 day + training ~1 day | Precision; target ≥ 50% at ≥ 50% recall |
| V4 | Read the two pending papers (CoT faithfulness; data selection for VLM fine-tuning) and turn one idea into a test | — | ~1 day | — |

### Orchestration (stage 2 + VLM)

- **Use the VLM as a re-checker, not a second detector.** Send it only the boxes where stage 2 is unsure:
  radar/missile boxes, and vehicle/arty/logistic boxes with radar or missile among their top-3 scores.
  - VLM agrees → keep; VLM rejects → drop or lower the score; VLM says radar/missile on a vehicle box → relabel.
- This aims directly at cause 1: of the 15 targets only the VLM got right where YOLO had a box, YOLO had called 7 of them vehicle.
- Success check: end-to-end radar/missile F1 on Test116 against stage 2 alone, **counting the new false positives** (the 60.5% union is a ceiling without them).
- Needs V1 + V2 first; the rule itself is CPU work on saved outputs.

### Suggested order

1. **This week, no GPU:** S1, S2, S3, V1 — cheap, and they decide what the GPU should do.
2. **Next GPU block:** S4 (+ S5 as the second seed of the same run), V2.
3. **Then:** V3 → orchestration rule → S6/S7 for the deployable package.

---

## 6. Evidence

- Deployment selection and weights: [knowledge/best_combination_ai.md §2](../../../knowledge/best_combination_ai.md).
- Error causes: `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/ERROR_CAUSES.md` and `error_causes.json`.
- Subset loop: [loop_status_ai.md](experiments/EXP-2026-10-28-01-train-subset-influence-loop/loop_status_ai.md).
- VLM metrics: `/sfs/vlm_qwen/experiments/2026-10-19/09_c1_full_stage1_eval/reports/FULL_METRICS.md`, `08_class_only_fourway_sft/reports/COMPARISON.md`.
- VLM vs YOLO: `/sfs/vlm_qwen/experiment_obs/2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-01-vlm-stage2-overlap/record_ai.md`.
- Earlier plan with the D/R correction: [pipeline_plan_ai.md](pipeline_plan_ai.md).
