# FINAL all-class model — ensemble (YOLOv8m-OBB + YOLO11l-OBB) + TTA + GE booster

Deployable 8-class oriented detector, **no SOC segregation** — the full july_w4 winning recipe on the
complete 27_08 data. Evaluated end-to-end on the **80-image test set** (Test1+Test2, leakage-free).

**Recipe:** GSD-normalize→0.30 m, tile 1024/20%, 8-band RGB=(5,3,2). Train = all up42 Train tiles
(10,758) + GE gepack booster (918 canvases / 35,682 objs). Two members (prod_y8m 68 ep, prod_y11l 78 ep).
Inference = **TTA (scales 1024 & 1536 × flips none,h) + noisy-or fusion + NMS 0.4**. Metric = recall-first
(max recall at precision ≥ 0.30), polygon IoU, scene-level.

> Numbers below are from `report_from_preds` (loads GT via `load_scene_objects`, correct). The sibling
> `infer_test` headline mis-loads GT for this tile layout and reported zeros — ignore it; predictions are fine.

## Headline (recall-first, P ≥ 0.30)
| | MICRO recall | MACRO recall |
|---|--:|--:|
| **IoU 0.3** | **74.4 %** | **58.0 %** |
| **IoU 0.5** | 72.4 % | 55.6 % |

## Per-class recall @ P ≥ 0.30
| class | GT | R@IoU0.3 | R@IoU0.5 | TP/FP/FN (IoU0.3) |
|---|--:|--:|--:|---|
| vehicle | 2041 | **91.6 %** | 90.5 % | 1595 / 888 / 446 |
| tank | 729 | **91.1 %** | 89.6 % | 507 / 239 / 222 |
| smallvehicle | 822 | 66.2 % | 63.3 % | 425 / 294 / 397 |
| radar | 176 | 57.4 % | 50.6 % | 68 / 27 / 108 |
| camo | 750 | 56.8 % | 53.3 % | 318 / 300 / 432 |
| missile | 162 | 41.4 % | 40.7 % | 67 / 139 / 95 |
| arty | 179 | 37.4 % | 34.6 % | 60 / 81 / 119 |
| logistic | 241 | 22.4 % | 22.0 % | 76 / 249 / 165 |

## Notes
- **Big classes are strong** (vehicle 91.6 %, tank 91.1 %) — the full 27_08 data fixed the class
  collapse the SOC-study single model showed (tank there was 5.6 %).
- **radar 57.4 % / missile 41.4 %** — the booster's rare-class objects help; radar notably clean
  (precision high, only 27 FP). Full confidence sweep in `FINAL_SCOREWISE.md`.
- **logistic (22 %) and arty (37 %) are the weak classes** — precision-limited (many FP), the next
  lever if these matter.

## Files (deploy artifacts)
- `prod_y8m_best.pt`, `prod_y11l_best.pt` — the two ensemble weights.
- `../predictions_final/labels/` — recall-first thresholded predictions (80 scenes, DOTA px in 0.30 m space).
- `../predictions_final_c05/labels/` — all boxes ≥ 0.05 (for your own thresholding).
- `FINAL_REPORT_DETAILED.md` — per-class P/R sweep, IoU 0.1/0.3/0.5, agnostic + detectability.
- `FINAL_SCOREWISE.md` — per-class TP/FP/FN + P/R/F1 at every confidence.
- Inference: `engine/infer_test.py --runs prod_y8m_s0,prod_y11l_s0 --scales 1024,1536 --flips none,h --nms 0.4 --conf 0.05 --fuse noisyor` (env `/sfs/envs/yolo_models`).
