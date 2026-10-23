# EXP-2026-10-23-01-best-per-class-oracle — AI-authored

Author: AI agent (Claude). Generated: 2026-10-23T18:49:31+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-23-01-best-per-class-oracle`

Status: completed (2026-10-23, analysis only — no training, no new inference).

## Request summary and authorization

2026-10-23: researcher asked for the best metrics achieved so far for all 8 classes, free to pick a different model / algorithm /
confidence threshold per class and per metric.

## Question and design

Upper bound of what our existing models already achieve on **test_80** (Test1 43 + Test2 37), IoU 0.3 (and 0.5), choosing per class the
best of 21 evaluated runs (m0001 + every Oct 14–15 run) and its best confidence threshold.
- **Official (P01)**: simple mapping, per-class confidence sweep from each run's `report_from_preds.py` report (classes scored independently).
- **Your rule set (P03)**: missile row (319 GT) and radar row (208 GT, stowed rule off / on), sweeping that class's threshold 0.20–0.99 with
  deploy post-processing (other classes at m0001 thresholds, cross-class dedup 0.4 with priority). Sweep validated: m0001 missile @0.85 = 67.1 / 72.1,
  radar @0.40 = 56.7 / 50.7 (exactly the published numbers).

## Actual setup and changes

Scripts: `/sfs/stage2/artifacts/EXP-2026-10-23-01-best-per-class-oracle/code/collect_p01.py`, `sweep_p03.py` (env `/sfs/envs/tide`).
Inputs: each run's `reports/P01_detailed.md` and `preds/c05/labels` (m0001: `FINAL/FINAL_REPORT_DETAILED.md`, `predictions_final_c05`).

## Results and failures

### A. Official scoring (P01), IoU 0.3 — best F1 per class
| class | best F1 | P | R | conf | model |
|---|--:|--:|--:|--:|---|
| arty | 38.3 | 42.8 | 34.6 | 0.90 | ens frozen-D y8m + y11l (EXP-2026-10-14-14) |
| camo | 48.3 | 53.1 | 44.3 | 0.70 | ens full-D (EXP-2026-10-14-16) |
| logistic | 28.6 | 26.7 | 30.7 | 0.60 | y11l on D (EXP-2026-10-14-13) |
| missile | 36.4 | 32.5 | 41.4 | 0.90 | m0001 ensemble |
| radar | 50.5 | 54.2 | 47.2 | 0.70 | ens control y8m + y11l (EXP-2026-10-14-09) |
| smallvehicle | 55.2 | 59.1 | 51.7 | 0.60 | m0001 ensemble |
| tank | 68.7 | 68.0 | 69.5 | 0.90 | m0001 ensemble |
| vehicle | 70.5 | 64.2 | 78.1 | 0.90 | m0001 ensemble |
| **MICRO** | **60.2** | 58.7 | 61.7 | | (best single run: ens frozen-D 59.9, m0001 59.7) |
| **MACRO** | **49.6** | 50.1 | 49.7 | | |

### B. Official scoring (P01), IoU 0.3 — best recall at precision ≥ 30 %
arty 37.4 (m0001) · camo 56.8 (m0001) · logistic 25.3 (ens full-D) · missile 42.0 (y8m C) · radar 60.8 (ens y11l + y11l-D) ·
smallvehicle 68.2 (ens R) · tank 91.1 (m0001) · vehicle 93.1 (ens B) → **MICRO 75.6 / MACRO 59.3** (m0001 alone: 74.4 / 58.0).

### C. IoU 0.5 (P01)
best F1 MICRO 59.1 / MACRO 48.1; R@P≥0.30 MICRO 73.4 / MACRO 56.6.

### D. Your rule set (P03), best threshold per model
| row | best F1 | R | P | threshold | model |
|---|--:|--:|--:|--:|---|
| missile (319) | **72.5** | 65.2 | 81.6 | 0.97 | ens C y8m + y11l (EXP-2026-10-14-07) |
| radar, stowed rule OFF (208) | **57.7** | 48.1 | 72.1 | 0.60 | original y11l alone (EXP-2026-10-14-17) |
| radar, stowed rule ON | **64.6** | 54.3 | 79.8 | 0.90 | ens D y8m + y11l (EXP-2026-10-14-08) |
References at today's deploy thresholds: m0001 missile 69.5, radar 53.2 / 56.1.

### E. Combined "operational" picture (8 classes: P01 best F1 for 6 classes + P03 missile + P03 radar)
arty 38.3 · camo 48.3 · logistic 28.6 · **missile 72.5** · **radar 57.7 (off) / 64.6 (on)** · smallvehicle 55.2 · tank 68.7 · vehicle 70.5
→ **MACRO F1 55.0 (radar rule off) / 55.8 (on)**. (No MICRO: missile/radar rows use different GT definitions than P01.)

## Conclusions and next steps

- These are **upper bounds**: model and threshold were chosen per class on the test set itself, so they are optimistic and would need a
  per-class mixture of up to 6 models to deploy. Confirming them needs a separate validation split (proposed for next week).
- The per-class pick adds little over m0001 on P01 (+0.5 F1 micro) — m0001 wins 4 of 8 classes; the real gains are in the **P03 rows**
  (missile 69.5 → 72.5, radar 53.2 → 57.7 / 64.6) and come mostly from **higher missile/radar thresholds** (0.97 / 0.60–0.90 vs 0.85 / 0.39).
- Weakest classes remain logistic (28.6) and arty (38.3).
- Scope: test_80 only; the 6 new 15_10 test scenes are not evaluated yet.
Evidence: [best_per_class_ai.json](best_per_class_ai.json), [p03_sweep_ai.json](p03_sweep_ai.json).

## Clarification — 2026-10-23 18:59 IST (researcher question: which rule per class, and which denominator holds camouflaged missiles)

- **Official one-to-one (P01, simple mapping):** P_Camo = Yes objects are counted as **camo** — the 145 camouflaged missiles and 30 camouflaged radars
  are in **camo's denominator (750)**, not missile's (162) or radar's (176). A `missile` box on a camouflaged missile is a missile **FP** and a camo **FN**.
- **P03 (researcher rules):** the missile row's denominator (319) = 307 `labels_detailed` missiles (incl. the 145 P_Camo) + 12 camo-parent objects with
  missile hardware; `missile` or `camo` both count as correct for those.
- **Correction to section E above:** it mixed P01 camo (750, which contains the 145 P_Camo missiles, 30 P_Camo radars and the 12 + 2 camo-parent
  missile/radar objects) with P03 missile/radar rows that contain the same objects — those ~189 objects were counted in two denominators.
  Section E's MACRO 55.0 / 55.8 is therefore not a consistent single scoring; use section A (fully one-to-one, official) or a P03 table with a
  rebuilt camo row (camo-only objects) for a consistent view.

## F. Per-class table with the rule-based (P03) missile and radar rows — 19:04 IST

The researcher asked for the table again, with the better missile and radar numbers. Six classes use official one-to-one scoring (P01). Missile and radar use P03 (camouflaged and empty-launcher credit, as above). Each per-class row is valid on its own. The MACRO mixes two scoring rules, so it is a summary and cannot be compared with section A.

| class | GT | F1 | Recall | Precision | conf | model |
|---|--:|--:|--:|--:|--:|---|
| arty | 179 | 38.3 | 34.6 | 42.8 | 0.90 | EXP-2026-10-14-14 |
| camo | 750 | 48.3 | 44.3 | 53.1 | 0.70 | EXP-2026-10-14-16 |
| logistic | 241 | 28.6 | 30.7 | 26.7 | 0.60 | EXP-2026-10-14-13 |
| **missile (P03)** | 319 | **72.5** | 65.2 (208 found) | 81.6 (151 of 185 boxes) | 0.97 | ens C y8m + y11l (EXP-2026-10-14-07) |
| **radar (P03, stowed rule off)** | 208 | **57.7** | 48.1 (100 found) | 72.1 (93 of 129 boxes) | 0.60 | original y11l (EXP-2026-10-14-17) |
| **radar (P03, stowed rule on)** | 208 | **64.6** | 54.3 (113 found) | 79.8 (79 of 99 boxes) | 0.90 | ens D y8m + y11l (EXP-2026-10-14-08) |
| smallvehicle | 822 | 55.2 | 51.7 | 59.1 | 0.60 | m0001 |
| tank | 729 | 68.7 | 69.5 | 68.0 | 0.90 | m0001 |
| vehicle | 2041 | 70.5 | 78.1 | 64.2 | 0.90 | m0001 |

**MACRO F1: 55.0 with the stowed rule off, 55.8 with it on.** The official one-to-one MACRO (section A) is 49.6.

### Other operating points for missile and radar (P03), across all 21 models

| row | goal | Recall | Precision | F1 | conf | model |
|---|---|--:|--:|--:|--:|---|
| missile | best F1 | 65.2 | 81.6 | 72.5 | 0.97 | EXP-2026-10-14-07 |
| missile | highest recall with precision ≥ 50 | 72.4 | 50.7 | 59.6 | 0.50 | EXP-2026-10-14-06 |
| missile | highest precision with recall ≥ 50 | 50.8 | 97.8 | 66.9 | 0.95 | y11l-D (EXP-2026-10-14-13) |
| missile | highest recall with precision ≥ 30 | 75.5 | 31.4 | 44.4 | 0.20 | EXP-2026-10-14-09 |
| missile | m0001 at deploy threshold | 67.1 | 72.1 | 69.5 | 0.85 | m0001 |
| radar, stowed off | best F1 | 48.1 | 72.1 | 57.7 | 0.60 | EXP-2026-10-14-17 |
| radar, stowed off | highest recall with precision ≥ 50 | 57.2 | 50.0 | 53.4 | 0.25 | EXP-2026-10-14-17 |
| radar, stowed off | highest recall with precision ≥ 30 | 63.0 | 31.9 | 42.3 | 0.20 | EXP-2026-10-14-14 |
| radar, stowed off | m0001 at deploy threshold | 56.7 | 50.7 | 53.5 | 0.40 | m0001 |
| radar, stowed on | best F1 | 54.3 | 79.8 | 64.6 | 0.90 | EXP-2026-10-14-08 |
| radar, stowed on | highest precision with recall ≥ 50 | 51.0 | 80.2 | 62.3 | 0.90 | EXP-2026-10-14-15 |
| radar, stowed on | highest recall with precision ≥ 50 | 64.9 | 50.0 | 56.5 | 0.25 | EXP-2026-10-14-17 |
| radar, stowed on | highest recall with precision ≥ 30 | 70.7 | 36.1 | 47.8 | 0.20 | EXP-2026-10-15-01 |
| radar, stowed on | m0001 at deploy threshold | 63.9 | 50.7 | 56.5 | 0.40 | m0001 |

**Caveats**
- These are upper bounds. The model and threshold for each row were chosen on test_80 itself.
- The 0.20 rows sit at the bottom of the threshold sweep, so recall at precision ≥ 30 may be slightly higher at lower thresholds.
- Under P03, the missile and radar rows also count the camouflaged missiles and radars that sit in camo's 750. Do not add up TP/FP across the table.

Evidence: [p03_sweep_ai.json](p03_sweep_ai.json).
