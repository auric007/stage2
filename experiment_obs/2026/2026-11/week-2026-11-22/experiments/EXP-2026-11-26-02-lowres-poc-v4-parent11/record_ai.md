# EXP-2026-11-26-02-lowres-poc-v4-parent11 — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-26_ai.md](../../daily/2026-11-26_ai.md).
Builds on: [EXP-2026-11-25-01](../EXP-2026-11-25-01-lowres-poc-v3/record_ai.md) v3 (recipe, tiles via v1), [EXP-2026-11-26-01](../EXP-2026-11-26-01-parent11-remap-eval/record_ai.md) (remap scorer, bar to beat).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-26-02-lowres-poc-v4-parent11`

Status: **running** — y11l-v4 training started 2026-11-26 19:29 IST (runner `code/run_v4.sh`, log `logs/pipeline.log`).

## Request summary and authorization
- Researcher, 2026-11-26 evening: retrain with the Parent-11 remap (`/sfs/stage2/14_PARENT11_ONEOFF_INSTANCE_REMAPPING_20261126.md`) wherever
  feasible, plan from the v3 child confusion matrix, aim = best test result after remapping. Approved (~19:00 IST): aerial trained; SPA oversampling;
  arty background penalty. Training order not chosen → sequential y11l → y8m (the recommended default).
- Researcher ~19:25 IST (during the smoke test): **P_Camo=Yes objects stay in their own class, not camo** → the dual-valid camo loss that had been
  built was removed before training; scoring adds a STRICT variant (P_Camo=Yes objects count only for their own class) next to the doc rule.

## Setup
- Data `code/make_v4_set.py`: the v1 tiles (EXP-2026-11-23-01 data/tiles; same images, grid, 40 % keep rule, Delete handling) with labels regenerated per
  object by `code/remap.py` (same code as the scoring). Classes: vehicle, smallvehicle, camo, missile, tank, radar, arty, aerial.
  Train transitions (836 scenes): logistic 3,967 → vehicle, ICV 1,650 tank → vehicle, empty / other PC02/PC07 1,039 missile → vehicle, SPA 640 → tank,
  PC01 EWS 118 → vehicle; aerial 577 added. Sanity: 23,073 tiles, v4 rows minus aerial = v1 rows on every tile.
  Train set = v1 base tiles 14,483 + 16 aerial-only tiles; repeats: RFS t = 0.15 recomputed (tank 2.40, arty 3.12, aerial 3.18), floors radar 3,
  missile 2.5, SPA 3 (new) → 19,885 tiles / epoch; instances / epoch vehicle 109,879 · smallvehicle 31,713 · camo 22,715 · missile 9,394 · tank 8,579
  (SPA 3,630) · radar 7,542 · arty 5,374 · aerial 2,978. Val = the same 4,449 test tiles (best.pt selection), labels remapped.
- Loss `code/cal_loss.py`: v3 confusion-aware BCE translated — positive radar 2.0 / missile 2.0 / tank 1.5; pairs radar↔vehicle/missile, missile↔vehicle/
  radar/arty/tank, tank↔camo 1.25; vehicle↔logistic pairs dropped; background tank 2.0 and **arty 2.0 (new)**. Fitness 0.5 mAP50 + 0.2 mAP50-95 + 0.2 mean
  AP50(radar, missile) + 0.1 AP50(tank). Training recipe = v3 (`code/train_v4.py`; 45 epochs, patience 15, imgsz 1024). Smoke test (1 %, 1 epoch) OK.
- Scoring `code/score_remap.py` (EXP-2026-11-26-01 copy + AERIAL=1: aerial thresholded and in MICRO(8)/MACRO(8); CAMO_ALT=0: STRICT). Points: F1-optimal,
  recall-heavy (P ≥ 30), P50 (vehicle / smallvehicle / camo) + P40 others. `code/hybrid.py` = per-class best model over v4 + v1–v3 (re-scored in both
  variants by `code/run_baselines.sh` → `reports/base_*`).
- Bar to beat (doc rule, IoU 0.3, F1-opt): v3 y11l MICRO(7) 67.4 / MACRO(7) 51.3; best existing per class 68.5 / 54.2.

## Results
(pending)
