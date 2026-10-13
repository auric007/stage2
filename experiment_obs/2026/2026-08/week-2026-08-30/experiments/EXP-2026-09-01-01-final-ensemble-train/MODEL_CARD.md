# m0001 · sep_w1 final ensemble (current BEST)

| | |
|---|---|
| trained in | [`experiments/2026-09/w1/2026-09-01_e0001_final-ensemble-train`](../../experiments/2026-09/w1/2026-09-01_e0001_final-ensemble-train) (2026-09-01) |
| members | YOLOv8m-OBB `prod_y8m_best.pt` (68 ep) + YOLO11l-OBB `prod_y11l_best.pt` (78 ep) — symlinks into the experiment's `FINAL/` |
| classes | arty, camo, logistic, missile, radar, smallvehicle, tank, vehicle |
| training data | `data/derived/train_final` = 27_08 Train tiles (10,758) + GE booster (918 canvases); 0.30 m GSD, 1024 tiles / 20 % overlap |
| inference | TTA scales 1024, 1536 × flips none, h · per-tile noisy-or → scene max · NMS 0.4 |
| deploy thresholds | radar 0.39 · missile 0.85 · arty 0.86 · logistic 0.77 · camo 0.42 · smallvehicle 0.24 · tank 0.54 · vehicle 0.84 · dedup IoU 0.4, priority radar, missile, arty, logistic |
| standalone package | `experiments/2026-09/w1/2026-09-01_e0001_final-ensemble-train/infer_point_stage2/` (infer.py, config.yaml, weights, README) |

## Metrics (80-image test, IoU 0.3)
| protocol | P % | R % | F1 % | note |
|---|--:|--:|--:|---|
| P01 recall-first (P ≥ 0.30) | — | MICRO 74.4 / MACRO 58.0 | — | `FINAL/FINAL_ENSEMBLE.md` |
| P01 deploy thresholds | 53.6 | 60.0 | 56.6 | `predictions_deploy/STATS.md` |
| P02 camo credit | 57.6 | 64.4 | 60.8 | e0006 |
| P03 missile row (draft) | 72.1 | 67.1 | 69.5 | e0007 `missile_rule_v2.py Yes,Nil` |
