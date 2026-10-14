# Week of 2026-10-11 - Review — AI-authored

Author: AI agent (Claude). Human/legacy companion: [review.md](review.md).
Moved here on 2026-10-14 from `review.md` under the human/AI authorship rule (content unchanged except links).

## Update — 2026-10-14 01:40 IST (Claude)

Recorded research work this week: [EXP-2026-10-12-01](experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md)
(running, Oct 12–14): attribute × prediction pattern study → user rule set P03. Missile row locked
(R 67.1 % / P 72.1 %); radar row analysed (R 56.7 % / P 50.0 %), decisions pending. On Oct 13 the stage2 research home was created
and all earlier work imported; on Oct 14 tracking moved into this notebook ([daily](daily/2026-10-14_ai.md)).

## Overnight GPU queue — 2026-10-14 02:13–08:25 IST (Claude)

All 9 jobs completed, no failures (V100; [daily log](daily/2026-10-14_ai.md)). Missile P03 = researcher's rule set; F1 in %.

| run | form | deploy P01 F1 | recall-first micro / macro | missile P03 R / P (F1) | radar F1 |
|---|---|--:|--:|--:|--:|
| [05 baseline](experiments/EXP-2026-10-14-05-baseline-y8m-single/record_ai.md) | single y8m | 57.1 | 71.6 / 53.9 | 54.2 / 78.2 (64.0) | 50.6 |
| [04 control](experiments/EXP-2026-10-14-04-control-finetune-a-unchanged/record_ai.md) | single, +10 ep | 55.0 | 70.4 / 52.7 | 66.1 / 65.1 (65.6) | 53.0 |
| [01 B](experiments/EXP-2026-10-14-01-relabel-b-nonlauncher-vehicle/record_ai.md) | single, +10 ep | 55.6 | 70.1 / 52.1 | 64.6 / 70.7 (67.5) | 53.1 |
| [02 C](experiments/EXP-2026-10-14-02-relabel-c-unloaded-vehicle/record_ai.md) | single, +10 ep | 55.0 | 69.9 / 52.1 | 64.9 / 72.5 (68.5) | 52.3 |
| [03 D](experiments/EXP-2026-10-14-03-relabel-d-both-vehicle/record_ai.md) | single, +10 ep | 55.7 | 70.5 / 53.3 | 60.5 / 77.2 (67.9) | 53.9 |
| [09 ens control](experiments/EXP-2026-10-14-09-ens-control-y8mft-y11l/record_ai.md) | y8m-ft + y11l | 56.0 | 73.4 / 52.8 | 69.0 / 61.6 (65.1) | 52.1 |
| [06 ens B](experiments/EXP-2026-10-14-06-ens-b-y8mft-y11l/record_ai.md) | y8m-B + y11l | 56.1 | 73.4 / 52.7 | 68.3 / 65.8 (67.1) | 52.5 |
| [07 ens C](experiments/EXP-2026-10-14-07-ens-c-y8mft-y11l/record_ai.md) | y8m-C + y11l | 55.8 | 73.4 / 52.6 | 68.0 / 65.5 (66.7) | 51.2 |
| [08 ens D](experiments/EXP-2026-10-14-08-ens-d-y8mft-y11l/record_ai.md) | y8m-D + y11l | **56.3** | **75.0 / 58.5** | 68.7 / 69.1 (**68.9**) | 52.8 |
| m0001 (Sept pipeline) | y8m + y11l | 56.6 | 74.4 / 58.0 | 67.1 / 72.1 (69.5) | 53.2 |

Readings: (1) extra fine-tuning alone raises missile recall but costs precision and ~2 points of overall F1; (2) every relabel beats its
control on missile F1 (single +1.9…+2.9, ensemble +1.6…+3.8) — D strongest, mostly via precision; (3) in deployable form, ensemble D is
level with m0001 overall (recall-first slightly higher, missile F1 −0.6) — a like-for-like m0001 run through the same pipeline is running
(EXP-2026-10-14-10). Failure: the GPU sat idle 08:25–15:55 because the agent's wake-up watcher was not re-armed after D.

## Radar stowed rule — evaluation only, all overnight models (Claude, 16:16 IST)

Researcher request 2026-10-14 ~16:10: score the same predictions with the stowed rule (Position Closed/Nil radar → `vehicle` accepted) OFF and ON. EWS→vehicle and helicopter-neutral were rejected (D-20261014-02), so neither is applied. 74 test radars have Position Closed/Nil. Script `code/engine_v1/score_rules.py` (rows `radar_base` / `radar_stowed`).

| run | radar, stowed rule OFF: R / P (F1) | radar, stowed rule ON: R / P (F1) | stowed radars predicted vehicle |
|---|--:|--:|--:|
| single baseline | 46.2 / 56.0 (50.6) | 53.8 / 56.0 (54.9) | 16 / 74 |
| control | 48.1 / 59.1 (53.0) | 55.3 / 59.1 (57.1) | 15 / 74 |
| B | 49.0 / 57.8 (53.1) | 56.7 / 57.8 (57.3) | 16 / 74 |
| C | 49.5 / 55.4 (52.3) | 57.2 / 55.4 (56.3) | 16 / 74 |
| D | 50.0 / 58.5 (53.9) | 58.2 / 58.5 (58.3) | 17 / 74 |
| ens control | 56.7 / 48.2 (52.1) | 63.0 / 48.2 (54.6) | 13 / 74 |
| ens B | 57.2 / 48.5 (52.5) | 63.5 / 48.5 (55.0) | 13 / 74 |
| ens C | 57.2 / 46.3 (51.2) | 63.5 / 45.9 (53.2) | 14 / 74 |
| ens D | 57.7 / 48.7 (52.8) | 64.9 / 48.3 (55.4) | 16 / 74 |
| m0001 (Sept pipeline) | 56.7 / 50.0 (53.2) | 63.9 / 50.0 (56.1) | 15 / 74 |

Reading: the rule adds ~7 points of radar recall to every model (13–17 of 74 stowed radars are predicted `vehicle`) and never changes radar precision (it only accepts a non-radar label). Best with rule ON: single D 58.3 F1; ensembles: m0001 56.1, ens D 55.4.

**TODO**: I want to where this table saved and also write down brieflty what this all rows exactly mean and where are their detailed results. like, i don't know whats mean of control, ensemle, mapping. so write down the briefly about each point down here

## Brief about all experiment

*Written by Claude, 2026-10-14 ~16:25 IST, in answer to the TODO above.*

### Where the tables are saved
- **Both tables live in this file** (`experiment_obs/2026/2026-10/week-2026-10-11/review_ai.md`): the overnight results table
  ("Overnight GPU queue" section) and the radar stowed-rule table (section just above this one).
- **Every row's full numbers** are inside that experiment's folder `experiments/EXP-…/`: `record_ai.md` (what was done, the results table at
  the bottom, conclusion), `metrics_ai.json` (all numbers), `P01_detailed_ai.md` (per-class precision/recall at every confidence),
  `deploy_STATS_ai.md` (per-class TP/FP at the deployment thresholds).
- **Weights, predictions and logs** (not synced, too big): `/sfs/stage2/artifacts/EXP-…/` → `runs/ft/weights/best.pt`, `preds/c05/` (every box ≥ 0.05),
  `preds/deploy/` (final deployed boxes), `logs/`, `reports/`.

### What the words mean
| term | meaning |
|---|---|
| **m0001** | Our current best model (Sept 2026): two detectors — **YOLOv8m** ("y8m", medium) and **YOLO11l** ("y11l", large) — whose predictions are combined. Weights in `/sfs/stage2/models/BEST/`. |
| **single model** | Only one detector (the y8m) is used. Faster, a bit weaker. |
| **ensemble** | Two detectors run on the same image and their scores are fused (if either is confident, the fused score is high). m0001 is an ensemble. "ensemble: D y8m + y11l" = the y8m trained on variant D + the original y11l. |
| **TTA** | Test-time augmentation: each image is also looked at at a larger size (1536 px) and mirrored; the four views are fused. Used in every row. |
| **fine-tune (+10 ep)** | Start from m0001's trained y8m and train it **10 more epochs** (passes over the training data) with a small learning rate. All overnight trainings are fine-tunes, because the V100 is too slow to train from zero overnight. |
| **baseline** | m0001's y8m **without any new training**, evaluated exactly like the others. The starting point. |
| **control** | Same 10-epoch fine-tune as B/C/D but with the **original, unchanged labels**. It tells us how much changes just because we trained 10 more epochs. A variant only "wins" if it beats the control. |
| **B / C / D (variants)** | Your missile relabel TODOs. **B**: missiles whose Functionality is not Launcher (Transporter, Transloader, Dump, empty) are trained as `vehicle`. **C**: Launchers with Is_Empty = Yes or Nil (canisters not loaded) are trained as `vehicle`. **D**: both B and C. Loaded launchers stay `missile`. |
| **R (variant)** | Radar relabel (queued): radars with Position Closed or Nil (stowed / not recorded) are trained as `vehicle`. |
| **mapping** | Which "true" class we compare against. **Simple mapping** (`labels_simple`): anything marked P_Camo = Yes counts as `camo`. **Detailed** (`labels_detailed`): the real hardware class (a camouflaged HQ9 is a missile). Your rule set uses the detailed labels. |
| **P01 / P02 / P03** | Scoring rule books in `/sfs/stage2/eval/protocols/`. **P01** = official: simple mapping, one exact class must match. **P02** = camo credit. **P03** = your rule set (missile row locked: camo OK for P_Camo/camo-parent missiles, `vehicle` OK when canisters not loaded). |
| **deploy F1** | All 8 classes together, P01, using the fixed deployment confidence thresholds of m0001 (radar 0.39, missile 0.85, …). F1 = balance of precision and recall. |
| **recall-first micro / macro** | For each class: the highest recall reachable while precision stays ≥ 30 %. **Micro** = averaged over all objects (big classes dominate); **macro** = average of the 8 class values (each class counts equally). |
| **missile (your rules) R / P (F1)** | The missile row under P03: R = share of the 319 test missiles found with an acceptable label; P = share of `missile` boxes that sit on a real missile. |
| **radar F1 / stowed rule OFF / ON** | Radar row (208 test radars). OFF = only `radar` (or `camo` if camouflaged) is correct. ON = for stowed radars (Position Closed/Nil, 74 of them) `vehicle` is also accepted. |
| **relabelled → vehicle** | Of the test objects a variant relabels, how many the model now calls `vehicle` (shows whether training on the new labels took effect). |

### Every row: what it is and where its details are
| row | what it is | details |
|---|---|---|
| single YOLOv8m baseline | m0001's y8m, no new training | [EXP-2026-10-14-05](experiments/EXP-2026-10-14-05-baseline-y8m-single/record_ai.md) |
| control (+10 ep, unchanged labels) | y8m fine-tuned 10 epochs on the original labels | [EXP-2026-10-14-04](experiments/EXP-2026-10-14-04-control-finetune-a-unchanged/record_ai.md) |
| B | y8m fine-tuned, non-launcher missiles → vehicle | [EXP-2026-10-14-01](experiments/EXP-2026-10-14-01-relabel-b-nonlauncher-vehicle/record_ai.md) |
| C | y8m fine-tuned, unloaded launchers → vehicle | [EXP-2026-10-14-02](experiments/EXP-2026-10-14-02-relabel-c-unloaded-vehicle/record_ai.md) |
| D | y8m fine-tuned, B + C | [EXP-2026-10-14-03](experiments/EXP-2026-10-14-03-relabel-d-both-vehicle/record_ai.md) |
| ensemble: control y8m + y11l | control y8m + original y11l | [EXP-2026-10-14-09](experiments/EXP-2026-10-14-09-ens-control-y8mft-y11l/record_ai.md) |
| ensemble: B / C / D | that variant's y8m + original y11l | [06 B](experiments/EXP-2026-10-14-06-ens-b-y8mft-y11l/record_ai.md) · [07 C](experiments/EXP-2026-10-14-07-ens-c-y8mft-y11l/record_ai.md) · [08 D](experiments/EXP-2026-10-14-08-ens-d-y8mft-y11l/record_ai.md) |
| m0001 (September pipeline) | the deployed model, numbers from September's evaluation run | [EXP-2026-09-01-01](../../2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record_ai.md); same-pipeline re-check running: [EXP-2026-10-14-10](experiments/EXP-2026-10-14-10-m0001-ens-same-pipeline/record_ai.md) |

**Queued now (results will be added here):** [11 D frozen backbone](experiments/EXP-2026-10-14-11-d-frozen-backbone/record_ai.md) ·
[12 radar R](experiments/EXP-2026-10-14-12-radar-r-stowed-vehicle/record_ai.md) · [13 y11l fine-tuned on D](experiments/EXP-2026-10-14-13-y11l-finetune-d/record_ai.md) ·
ensembles [14](experiments/EXP-2026-10-14-14-ens-dfrozen-y11l/record_ai.md), [15](experiments/EXP-2026-10-14-15-ens-r-y8mft-y11l/record_ai.md), [16 full D](experiments/EXP-2026-10-14-16-ens-d-full-y8md-y11ld/record_ai.md).

**In one line:** the variants are compared with the **control** to see the effect of the relabel; the **ensemble** rows show the deployable form;
**m0001** is the bar to beat.

## Follow-up queue — 2026-10-14 15:56–21:19 IST (Claude)

All 7 jobs completed, no failures. m0001 deploy thresholds, IoU 0.3, F1 in %; radar "off/on" = stowed rule.

| run | form | deploy F1 | missile P03 F1 (R / P) | radar F1 off / on |
|---|---|--:|--:|--:|
| [10 m0001](experiments/EXP-2026-10-14-10-m0001-ens-same-pipeline/record_ai.md) | ensemble, same pipeline (reproduces Sept exactly) | **56.6** | 69.5 (67.1 / 72.1) | 53.2 / 56.1 |
| [11 D frozen backbone](experiments/EXP-2026-10-14-11-d-frozen-backbone/record_ai.md) | single y8m | 56.3 | 65.2 (52.4 / 86.3) | 48.3 / 52.2 |
| [12 radar R](experiments/EXP-2026-10-14-12-radar-r-stowed-vehicle/record_ai.md) | single y8m | 55.7 | 66.3 (66.1 / 66.5) | 53.6 / **58.7** |
| [13 y11l on D](experiments/EXP-2026-10-14-13-y11l-finetune-d/record_ai.md) | **single y11l** | **56.7** | **71.5 (59.6 / 89.3)** | 52.2 / 56.3 |
| [14 ens frozen-D](experiments/EXP-2026-10-14-14-ens-dfrozen-y11l/record_ai.md) | y8m-Dfrozen + y11l | 55.3 | 70.1 (64.6 / 76.6) | 49.6 / 51.9 |
| [15 ens R](experiments/EXP-2026-10-14-15-ens-r-y8mft-y11l/record_ai.md) | y8m-R + y11l | 55.7 | 65.9 (69.3 / 62.8) | 51.5 / 54.5 |
| [16 ens full D](experiments/EXP-2026-10-14-16-ens-d-full-y8md-y11ld/record_ai.md) | y8m-D + y11l-D | 55.7 | 70.0 (68.0 / 72.1) | 50.9 / 53.6 |

Readings: (1) the single y11l fine-tuned on D is the best model so far on deploy F1 and missile F1 (beats the m0001 ensemble);
(2) ensembles lose radar precision at m0001's thresholds (noisy-or inflates radar scores) — thresholds must be re-tuned per model for a fair
ensemble comparison; (3) radar R helps only where the radar-predicting model was trained on R.
Queued next (21:20): [17 y11l alone](experiments/EXP-2026-10-14-17-y11l-single-baseline/record_ai.md), [18 y11l control fine-tune](experiments/EXP-2026-10-14-18-y11l-control-finetune-a/record_ai.md),
[19 y11l on D+R](experiments/EXP-2026-10-14-19-y11l-finetune-dr/record_ai.md).

## YOLO11l series — 2026-10-14 21:22 – 2026-10-15 01:04 IST (Claude)

| run (single YOLO11l) | deploy F1 | missile P03 F1 (R / P) | radar off R / P (F1) | radar on F1 |
|---|--:|--:|--:|--:|
| [17 original](experiments/EXP-2026-10-14-17-y11l-single-baseline/record_ai.md) | **57.8** | 70.1 (60.8 / 82.6) | 53.4 / 58.7 (55.9) | **59.8** |
| [18 control fine-tune](experiments/EXP-2026-10-14-18-y11l-control-finetune-a/record_ai.md) | 57.2 | 69.5 (62.1 / 78.8) | 50.0 / 51.9 (50.9) | 54.4 |
| [13 D](experiments/EXP-2026-10-14-13-y11l-finetune-d/record_ai.md) | 56.7 | **71.5** (59.6 / 89.3) | 51.0 / 53.6 (52.2) | 56.3 |
| [19 D+R](experiments/EXP-2026-10-14-19-y11l-finetune-dr/record_ai.md) | 55.7 | 66.8 (51.7 / 94.2) | 50.0 / **60.0** (54.5) | 58.8 |

**Correction (supersedes the "Follow-up queue" reading 1 above):** the best all-round model so far is the **original YOLO11l alone** (EXP-17),
not y11l-on-D; y11l-on-D is best only on the missile row. Fine-tuning (even on unchanged labels) costs overall F1 and radar; relabels add
missile/radar precision relative to their control. Queued next: [2026-10-15-01 ensemble y11l original + y11l-D](../week-2026-10-11/experiments/EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d/record_ai.md),
[2026-10-15-02 y11l on D, 4× lower LR](../week-2026-10-11/experiments/EXP-2026-10-15-02-y11l-finetune-d-lowlr/record_ai.md).
