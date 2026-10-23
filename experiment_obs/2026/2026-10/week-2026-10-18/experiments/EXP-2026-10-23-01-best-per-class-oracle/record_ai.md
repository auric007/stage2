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
