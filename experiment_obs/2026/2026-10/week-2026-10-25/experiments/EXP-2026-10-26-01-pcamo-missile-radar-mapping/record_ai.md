# EXP-2026-10-26-01-pcamo-missile-radar-mapping — AI-authored

Author: AI agent (Claude). Generated: 2026-10-26 16:11 IST.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-26-01-pcamo-missile-radar-mapping`

Status: done (analysis only, no GPU).

## Request summary and authorization

2026-10-26, about 16:05 IST. The researcher said:
- they had run the scorings where camouflaged missiles go under camo in one and under missile in the other;
- they half-remembered that camo gave the better result.

They asked:
- which mapping is best for camouflaged missiles, settled clearly;
- the same for radar.

Scope: evaluation only on saved predictions of the 21 models from EXP-2026-10-23-01/-02.

## Question and design

**Which objects:**
- **Camouflaged missile (157):**
  - 145 objects whose labels_detailed class is missile and P_Camo = Yes;
  - 12 camo-parent objects whose fine label is missile hardware (HQ9 6, MissileSystem 5, HQ22 1).
- **Camouflaged radar (32):**
  - 30 objects whose labels_detailed class is radar and P_Camo = Yes;
  - 2 camo-parent objects with a radar fine label.
- The official simple mapping puts all of them under **camo**.

**Training labels (checked):** every model scored here was trained with these objects in their hardware class:
- m0001 and all the fine-tunes;
- build_final / prep_soc use the labels_detailed class, `o["true"]`;
- variant A reproduces train_final exactly.

So camo for these objects exists only in the official evaluation. No training experiment ever put them under camo.

**Scoring:**
- the official P01 logic (report_from_preds.py: per-class greedy matching on c05 predictions, delete regions dropped, confidence 0.05–0.90, IoU 0.3);
- one class per object;
- **only the camouflaged missile/radar objects change**;
- everything else keeps the official mapping.

**Options:**
- 5 for missile × 3 for radar = 15 GT versions, each scored for m0001 alone and for the best per class across the 21 models.
- **Check:** the all-camo option equals the official P01 numbers for all 21 runs (0 TP/FP/FN mismatches).

**How the model labels these objects** (m0001, top-scoring class of the box on the object):
- **Camouflaged missiles:** missile 87, vehicle 42, camo 15, logistic 7, radar 3, no box 3. MLRS: 19 of 20 vehicle.
- **Camouflaged radars:** camo 15, radar 13, vehicle 2, no box 2.

## Results — missiles (camouflaged radars kept under camo), test_80, IoU 0.3

| where the 157 camouflaged missiles go | missile GT | m0001 missile F1 (P / R) | m0001 camo F1 | m0001 MICRO F1 / MACRO | best-per-class MICRO / MACRO |
|---|--:|---|--:|---|---|
| M1 all → camo (official) | 162 | 36.4 (32.5 / 41.4) | 46.5 | 59.7 / 49.0 | 60.2 / 49.6 |
| M2 all → missile | 319 | 59.4 (75.7 / 48.9) | 49.5 | 61.2 / 52.2 | 61.6 / 52.8 |
| M3 all → missile, MLRS → vehicle | 299 | 59.8 (73.3 / 50.5) | 49.5 | 61.5 / 52.4 | 61.9 / 52.9 |
| M4 loaded launchers → missile, rest → camo | 277 | 62.1 (72.8 / 54.2) | 47.9 | 61.1 / 52.4 | 61.6 / 52.9 |
| **M5 by fine type** (P04: HQ9/S300/HQ12/MissileSystem → missile, MLRS/HQ16 → vehicle, rest → camo) | 276 | 61.0 (71.4 / 53.3) | 49.7 | **61.6 / 52.5** | **62.0 / 53.0** |

What the table shows:
- **Under camo is the worst option.** It is worst for the missile row (36.4) and also for the camo row (46.5 against 49.5). The model calls most of these objects missile, so under camo they count against both rows.
- **Moving them out of camo gives almost all the gain:** MICRO +1.5 to +1.9, MACRO +3.2 to +3.5 (m0001).
- **The four "out of camo" options are within 0.5 MICRO / 0.3 MACRO of each other.**
  - M5 is the highest overall.
  - M3 is almost the same with one simple rule (MLRS → vehicle).
  - M4 gives the best missile row on its own but a weaker camo row.

## Results — radars (missiles at M5), test_80, IoU 0.3

| where the 32 camouflaged radars go | radar GT | m0001 radar F1 (P / R) | m0001 camo F1 | m0001 MICRO F1 (R) / MACRO | best radar F1 | best MICRO / MACRO |
|---|--:|---|--:|---|--:|---|
| R1 all → camo (official) | 176 | 50.2 (71.6 / 38.6) | 49.7 | 61.6 (62.4) / 52.5 | 50.5 | 62.0 / 53.0 |
| **R2 all → radar** | 208 | 54.8 (61.3 / 49.5) | 48.4 | 61.5 (62.8) / **52.9** | **56.7** | **62.1 / 53.7** |
| R3 Position Open → camo, rest → radar | 185 | 51.0 (53.6 / 48.6) | 49.4 | 61.5 (62.8) / 52.6 | 52.9 | 62.0 / 53.3 |

What the table shows:
- **Under radar is better.** The radar row gains about 4.6 F1 (m0001) and 6.2 (best per class).
- The camo row loses about 1.2.
- MICRO is unchanged to +0.1 and MICRO recall gains +0.4.
- The split by Position is worse than both.

**On top of the full P04 rule set** (14 rules, EXP-2026-10-23-02) plus camouflaged radar → radar (30 objects; the 2 camo-parent radars stay camo):

| | m0001 MICRO F1 / R / MACRO | best-per-class MICRO F1 / R / MACRO | R@P≥0.30 MICRO / MACRO (best) | radar F1 (best) |
|---|---|---|---|--:|
| P04, 14 rules | 63.0 / 63.9 / 53.8 | 63.5 / 63.6 / 54.4 | 79.2 / 64.9 | 53.3 |
| **P04 + camouflaged radar → radar** | **63.0 / 64.3 / 54.0** | **63.6 / 63.9 / 55.0** | **79.3 / 65.5** | **58.8** |

## Supporting evidence

- [summary_ai.json](summary_ai.json): all 15 options, m0001 alone and best per class, IoU 0.3 and 0.5.
- [rules_p04v2_ai.json](rules_p04v2_ai.json): the 14 P04 rules + camouflaged radar → radar.
- Code (artifacts, `code/`):
  - `compare_mapping.py` (the 15 options);
  - `summarize.py`;
  - `score_remap.py` (a copy from EXP-2026-10-23-02, reading this experiment's rules).
- Environment: `/sfs/envs/yolo_models/bin/python`. The first run was killed when the session ended; the rerun took 61 s.

## Conclusions and next steps

- **Missile: under missile is clearly better than under camo.** The opposite recollection doesn't hold on these models.
- **Recommended missile mapping:**
  - M5 (by fine type, as in P04) if the best number matters;
  - M3 (all → missile, MLRS → vehicle) if one simple rule is preferred.
  - The difference between them is ≤ 0.1 MICRO / ≤ 0.1 MACRO.
- **Radar: camouflaged radars → radar.** It is best for the radar row and for MACRO, and costs about 1.2 F1 on camo.
- **Best numbers so far (upper bound):** P04 + camouflaged radar → radar, best per class MICRO F1 63.6 / MACRO 55.0. m0001 alone gets 63.0 / 54.0.
- **Limitation:** the rules and the picks were made on test_80. This holds for models trained with these objects in their hardware class. A model trained with them as camo could behave differently; that was never trained.
- Decision recorded as D-20261026-01 (P04 v2 draft, pending the researcher's review).
