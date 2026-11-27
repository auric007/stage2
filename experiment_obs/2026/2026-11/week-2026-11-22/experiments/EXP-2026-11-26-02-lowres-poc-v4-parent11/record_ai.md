# EXP-2026-11-26-02-lowres-poc-v4-parent11 — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-26_ai.md](../../daily/2026-11-26_ai.md).
Builds on: [EXP-2026-11-25-01](../EXP-2026-11-25-01-lowres-poc-v3/record_ai.md) v3 (recipe, tiles via v1), [EXP-2026-11-26-01](../EXP-2026-11-26-01-parent11-remap-eval/record_ai.md) (remap scorer, bar to beat).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-26-02-lowres-poc-v4-parent11`

Status: **completed** 2026-11-27 03:50 IST (y11l-v4 19:47 → 00:48, 34 epochs, best epoch 23 by mAP50; y8m-v4 00:56 → 03:42, 27 epochs, best 23; runner `code/run_v4.sh`).
First start 19:29 IST stopped at epoch 2 on researcher request (artefacts in `_aborted_136/`).

## Request summary and authorization
- Researcher, 2026-11-26 evening: retrain with the Parent-11 remap (`/sfs/stage2/14_PARENT11_ONEOFF_INSTANCE_REMAPPING_20261126.md`) wherever
  feasible, plan from the v3 child confusion matrix, aim = best test result after remapping. Approved (~19:00 IST): aerial trained; SPA oversampling;
  arty background penalty. Training order not chosen → sequential y11l → y8m (the recommended default).
- Researcher ~19:25 IST (during the smoke test): **P_Camo=Yes objects stay in their own class, not camo** → the dual-valid camo loss that had been
  built was removed before training; scoring adds a STRICT variant (P_Camo=Yes objects count only for their own class) next to the doc rule.

- Researcher ~19:45 IST: remove the 51 zero-annotation UP42 scenes (`/sfs/stage2/up42_zero_annotation_images_20261126.md`; 36 in our train, 15 in
  our test, 0 labels in our labels_simple) from train and test and start again → **v4 test = 121 scenes** (protocol change for this experiment;
  dataset files untouched). Train loses 402 background tiles (19,885 → 19,421 tiles / epoch, no objects lost); val 4,449 → 3,995 tiles.

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
  STRICT bar (v1–v3 re-scored 19:45 IST; doc-rule re-score reproduces EXP-2026-11-26-01 exactly): v3 y11l 67.2 / 50.9 (camo 68.1 vs 70.0, radar 61.1 vs 61.7; see
  `reports/base_v3_y11l__strict.md`); best existing per class 68.1 / 53.8 (`reports/hybrid_strict_base.md`). STRICT moves only camo (−2 to −3 F1) and radar / missile slightly.
  y11l-v4 epoch 1 running at 3.2 it/s (1,658 it / epoch ≈ 9 min + val).

- **121-scene bar** (v1–v3 re-scored with EXCL_STEMS → `reports/base121_*`, IoU 0.3, F1-opt, MICRO(7) / MACRO(7), doc rule | STRICT):
  v1 y11l 67.2 / 51.2 | 66.7 / 50.5 · v1 ens 66.3 / 49.5 | 66.1 / 49.1 · v2 y8m 66.4 / 49.4 | 65.8 / 48.8 · v2 y11l 67.1 / 53.4 | 66.7 / 53.0 ·
  v2 ens 68.0 / 53.3 | 67.5 / 52.8 · **v3 y11l 68.7 / 56.3 | 68.4 / 56.0**; best existing per class 69.4 / 57.4 | 69.0 / 57.0
  (`reports/hybrid_doc_base121.md`, `hybrid_strict_base121.md`; arty 51.9 from v3 now that the 185 false arty boxes of 25JUN16072810…879 are gone).

## Side analysis — radar / missile recall by SOC (researcher ~20:15 IST, existing models)
- `code/soc_recall.py` → `reports/soc_recall_radar_missile.md|json`. UP42: remap GT, strict, 121-scene confs; CartoSat: package classes, CartoSat-tuned confs. IoU 0.3.
- UP42 v3 y11l recall best-F1 / P≥30 / any box: radar Pixels (150) 57 / 71 / 78, Site (30) 37 / 53 / 53, Third_Party (23) 26 / 35 / 39; missile Pixels (172) 53 / 67 / 74, Site (20) 10 / 20 / 35, Third_Party (20) 35 / 55 / 55, SOC Nil (32) 3 / 44 / 56. Best non-Pixels missile at P≥30: v2 ens 34 / 72 (47 %).
- CartoSat (7 radar, 7 missile): no model boxes any of the 5 non-Pixels radars (Temporal 3, Third_Party 2) even at conf 0.05; Pixels radars 2 / 2 (v2 y8m). Missile all non-Pixels: Site MissileSystem 0 / 3, Temporal SPADA 0 / 1, Third_Party HQ16 up to 2 / 3 (v1 y8m, v1 ens).

## Results
Pipeline ended 2026-11-27 03:50 IST (y11l-v4 best epoch of 35 run, y8m-v4 of 28; TTA inference; remap scoring on the 121-scene test, EXCL_STEMS = zero_annotation_51; report headers still say "136" — hard-coded text, scenes are 121).
IoU 0.3, F1-opt, MICRO(7) / MACRO(7) · MICRO(8)/MACRO(8) incl. aerial, doc rule (strict in brackets):
- v4 y11l 68.0 / 50.3 · 67.9 / 52.5 (67.8 / 50.0) · v4 y8m 65.8 / 48.2 (65.4 / 47.7) · v4 ens 68.5 / 50.9 · 68.6 / 53.4 (68.3 / 50.5).
- Bar v3 y11l (121) 68.7 / 56.3 (aerial untrained). v4 does not beat it on MICRO(7); MACRO(7) −5.4 driven by arty: v4 y11l P 36.0 / R 13.8 / F1 20.0 (9 / 16 / 56) vs v3 69.2 / 41.5 / 51.9 (27 / 12 / 38) → arty background penalty over-suppressed recall.
- v4 gains: aerial 71.0 (ens; v3 untrained), vehicle 73.0 (y11l, +0.8), missile 51.9 (y11l, +2.1), tank 24.6–26.8 (+1.8–4.0). Losses: camo 63.7–67.4 (v3 70.1), radar 56.7–60.7 (v3 61.9), smallvehicle 63.9 (v3 65.7).
- Per-class HYBRID v1–v4 (doc): MICRO(7) P 72.5 / R 67.0 / F1 69.6 · MACRO(7) 57.5 · MACRO(8) 59.2 (prev best existing 69.4 / 57.4); v4 contributes vehicle, missile (y11l), tank (y8m), aerial (ens).
