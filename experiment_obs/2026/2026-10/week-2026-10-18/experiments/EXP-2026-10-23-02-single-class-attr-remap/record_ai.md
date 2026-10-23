# EXP-2026-10-23-02-single-class-attr-remap — AI-authored

Author: AI agent (Claude). Generated: 2026-10-23T19:15:32+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-23-02-single-class-attr-remap`

Status: done (analysis only, no GPU).

## Request summary and authorization

2026-10-23, about 19:05 IST. The researcher asked to:
- drop P03 (several accepted classes per object);
- report the best metrics so far;
- allowed mapping any attribute to any class if it raises recall, but with **exactly one class per object**.

Scope: evaluation only, on saved predictions of the 21 models already scored in [EXP-2026-10-23-01](../EXP-2026-10-23-01-best-per-class-oracle/record_ai.md). Nothing was trained and nothing new was inferred.

## Question and design

**Question:** which one-class-per-object attribute remap gives the most recall, and what are the best per-class metrics under it?

**How the rules were chosen**
- **Evidence:** for each test object, the highest m0001 c05 box score of each class at IoU ≥ 0.3 (instances.tsv). An object counts as "found as class X" when that score reaches m0001's official best-F1 confidence for X.
- **Groups:** (official class, detailed class), split by the fine type, by Functionality and Is_Empty, or by Position.
- **Greedy selection:** each step remaps the group with the largest recall gain (objects found as the new class minus objects found as the current class).
- **Constraints:** a group needs at least 5 objects, a gain of at least 3, and a gain of at least 10% of the group. A whole official class may not be remapped, because that would delete a class.
- **Order:** three orders were tried — largest gain first (gain 153), fine type first (164) and attributes first (149). Fine type first was kept.

**Scoring:** the official P01 logic from report_from_preds.py:
- per-class greedy matching on c05 predictions, with delete regions dropped;
- the same confidence grid, 0.05 to 0.90;
- IoU 0.3 and 0.5;
- the only change is that each ground-truth object's class comes from the remap.

**Check:** the identity mapping reproduces the official P01 TP/FP/FN and R@P≥0.30 for all 21 runs × 8 classes × 2 IoU gates (336 of 336 rows).

## The 14 remap rules (each object gets one class; the first matching rule applies)

| rule | objects (official class → detailed class, attribute) | new class | n | found as old → new class (m0001) |
|---|---|---|--:|--:|
| R01 | camouflaged HQ9 missile (official camo) | missile | 57 | 7 → 43 |
| R02 | MLRS (official missile) | vehicle | 28 | 3 → 25 |
| R03 | camouflaged MLRS (official camo) | vehicle | 20 | 0 → 20 |
| R04 | camouflaged S300 (official camo) | missile | 22 | 0 → 16 |
| R05 | EWS radar | vehicle | 16 | 0 → 11 |
| R06 | SPATracked (arty) | tank | 17 | 1 → 11 |
| R07 | camo-parent LTV | vehicle | 23 | 7 → 16 |
| R08 | HQ9 missile, not camouflaged | vehicle | 34 | 15 → 24 |
| R09 | camouflaged HQ12 | missile | 17 | 3 → 11 |
| R10 | camouflaged MissileSystem | missile | 18 | 8 → 15 |
| R11 | camouflaged tank (fine Tank) | tank | 8 | 1 → 8 |
| R12 | camouflaged HQ16 | vehicle | 7 | 1 → 4 |
| R13 | MissileSystem missile, not camouflaged | vehicle | 17 | 2 → 5 |
| R14 | ABLV (tank) | vehicle | 10 | 2 → 5 |

In total, 294 objects move class:

| move | objects |
|---|--:|
| camo → missile | 114 |
| missile → vehicle | 79 |
| camo → vehicle | 50 |
| arty → tank | 17 |
| radar → vehicle | 16 |
| tank → vehicle | 10 |
| camo → tank | 8 |

**Marginal groups, not applied** (gain below 10% of the group):
- camo-parent ICV → tank: 88 → 94 of 195.
- RadarSystem → vehicle: 21 → 26 of 76.

**Logistic → vehicle was not used.** It is the whole logistic class, so it would remove a class.

**Drop-one ablation:** removing any one rule lowers recall or F1, by −0.1 to −0.8 micro recall. So every rule helps.

## Results — test_80, IoU 0.3, free choice of model and confidence per class (upper bound)

| class | GT official → remap | F1 | Precision | Recall | TP / FP / FN | conf | model | R@P≥0.30 (best model) |
|---|--:|--:|--:|--:|---|--:|---|--:|
| arty | 179 → 162 | 39.7 | 42.1 | 37.7 | 61 / 84 / 101 | 0.90 | ens D-frozen + y11l (EXP-2026-10-14-14) | 40.7 (m0001) |
| camo | 750 → 578 | 51.2 | 53.4 | 49.1 | 284 / 248 / 294 | 0.80 | EXP-2026-10-14-14 | 64.4 (EXP-2026-10-14-08) |
| logistic | 241 | 28.6 | 26.7 | 30.7 | 74 / 203 / 167 | 0.60 | y11l-D (EXP-2026-10-14-13) | 25.3 (EXP-2026-10-14-16) |
| missile | 162 → 197 | 64.2 | 66.7 | 61.9 | 122 / 61 / 75 | 0.90 | EXP-2026-10-14-14 | 72.6 (EXP-2026-10-14-16) |
| radar | 176 → 160 | 53.3 | 71.6 | 42.5 | 68 / 27 / 92 | 0.90 | m0001 | 62.5 (EXP-2026-10-14-09) |
| smallvehicle | 822 | 55.2 | 62.5 | 49.4 | 406 / 244 / 416 | 0.50 | EXP-2026-10-14-13 | 68.2 (EXP-2026-10-14-15) |
| tank | 729 → 744 | 70.3 | 70.2 | 70.4 | 524 / 222 / 220 | 0.90 | m0001 | 91.9 (m0001) |
| vehicle | 2041 → 2196 | 72.9 | 68.7 | 77.6 | 1705 / 778 / 491 | 0.90 | m0001 | 93.5 (EXP-2026-10-14-16) |

Summary rows. Every row is one-to-one and uses one class per object, so MICRO is valid here (no double counting).

| scoring | MICRO F1 (P / R) | MACRO F1 (P / R) | R@P≥0.30 MICRO / MACRO |
|---|---|---|---|
| official P01, best per class | 60.2 (59.3 / 61.2) | 49.6 (51.1 / 49.0) | 75.6 / 59.3 |
| **remap, best per class** | **63.5 (63.5 / 63.6)** | **54.4 (57.7 / 52.4)** | **79.2 / 64.9** |
| official P01, m0001 alone | 59.7 (58.4 / 61.1) | 49.0 (51.6 / 48.4) | 74.4 / 58.0 |
| remap, m0001 alone (one deployable model) | 63.0 (62.2 / 63.9) | 53.8 (56.1 / 52.8) | 78.4 / 63.8 |

Results at IoU 0.5:

| scoring | MICRO F1 | MACRO F1 | R@P≥0.30 MICRO / MACRO |
|---|--:|--:|---|
| remap, best per class | 62.7 | 53.1 | 77.2 / 62.3 |
| official P01, best per class | 59.1 | 48.1 | 73.4 / 56.6 |

## Supporting evidence

- [rules_ai.json](rules_ai.json): rules, evidence counts and the marginal groups.
- [summary_ai.json](summary_ai.json): best per class, m0001 alone, IoU 0.3 and 0.5, and the drop-one ablation.
- Code (artifacts, `code/`): `select_rules.py`, `score_remap.py` (identity / remap / `--drop Rxx`) and `summarize.py`.
- Per-run JSON is in `reports/`.
- Environment: `/sfs/envs/yolo_models/bin/python` (score_remap.py, summarize.py) and `/sfs/envs/tide/bin/python` (select_rules.py).

## Conclusions and next steps

- With one class per object, the remap raises best-per-class MICRO F1 from 60.2 to 63.5 and MACRO from 49.6 to 54.4. MICRO recall goes 61.2 → 63.6, and R@P≥0.30 MICRO 75.6 → 79.2.
- **Missile gains the most: F1 36.4 → 64.2.**
  - Camouflaged HQ9, S300, HQ12 and MissileSystem count as missile.
  - MLRS (camouflaged or not), uncamouflaged HQ9, uncamouflaged MissileSystem and camouflaged HQ16 count as vehicle.
- These numbers are optimistic in two ways:
  - The rules were chosen from m0001's behaviour on test_80 itself.
  - Model and confidence were picked per class on the same test set.
- Confirming them needs a held-out split.
- The single-model row (m0001 63.0 / 53.8) is the number one deployable model reaches under these rules.
- This replaces P03 for reporting at the researcher's request ([D-20261023-01](../../../../../knowledge/decisions_ai.md)).
- **Open:** the researcher should review the 14 rules for semantic sense, especially R08 and R13. Those send launchers that are not camouflaged to vehicle by fine type rather than by Is_Empty.
