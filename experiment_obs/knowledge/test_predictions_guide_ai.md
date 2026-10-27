# Test-set predictions guide (stage 2, test_80) — AI-authored

Author: AI agent (Claude). Written 2026-10-26. Built in [EXP-2026-10-26-02](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-02-best-pred-files-guide/record_ai.md).
Human companion: none. Write your notes in a separate file without `_ai`.

This guide shows where every prediction file for the 80 test images is, together with the ground truth, the images and the scoring scripts. It gives the file formats and our findings so far. Use it to compare the best predictions image by image.

## 1. Quick start: what to compare

| want | go to |
|---|---|
| **The best boxes we have, for each of the 80 images.** Each class comes from its best model; this is an upper bound. | `/sfs/stage2/artifacts/EXP-2026-10-26-02-best-pred-files-guide/preds/best_per_class/labels/<scene>.txt` |
| **The best deployable single-model output** (m0001, deploy thresholds + dedup) | `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/predictions_deploy/labels/<scene>.txt` |
| **Ground truth in the same coordinates and the current class mapping (P04 v2)**, with instance IDs | `/sfs/stage2/artifacts/EXP-2026-10-26-02-best-pred-files-guide/gt_p04v2/labels/<scene>.txt` |
| **Per image × class GT / TP / FP / FN for both sets** above | [per_image_ai.tsv](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-02-best-pred-files-guide/per_image_ai.tsv) |
| **Raw boxes of any model**, all scores ≥ 0.05 | section 4 |
| **Per-image deploy reports** (confusion matrix, every instance by ID) | `/sfs/stage2/artifacts/EXP-2026-10-08-01-imagewise-deploy/SUMMARY.md` and `images/<II>_<scene>.md` (P01 official mapping) |

All three files (best set, deploy set, GT) use the **same coordinate space**, so they can be overlaid or matched directly.

## 2. Test set

- **80 images.** Test1 has 43 and Test2 has 37. The scene list is `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/state/test_scenes.txt`, one stem per line. The frozen copy is `/sfs/stage2/eval/test_sets/test_80`.
- **Dataset:** `/sfs/data/up42_platform_data_27_08_with_soc/{Test1,Test2}/`. Read `/sfs/data/CLAUDE.md` before using any dataset.
- **Images** are native GeoTIFFs at `/sfs/data/up42_platform_data_27_08/images/<image>.tif`.
  - The `<split>/images` symlinks inside `_with_soc` are broken since the 2026-10-08 move. `common.scene_image()` falls back to the folder above.
  - Bands: 8-band uses RGB = bands 5,3,2; 4-band uses 3,2,1. See `common.read_image_rgb_bgr()`.
- **Per-scene manifest:** `.../EXP-2026-09-01-01-final-ensemble-train/state/gsd_norm.json`. It holds split, image file, `gsd` (native, m/px), `scale_to_target`, and native `W`, `H`.
- **5,100 GT objects** in the 8 trained classes: arty, camo, logistic, missile, radar, smallvehicle, tank, vehicle. Aerial, radome, ballistic and concealment are dropped, and spa is merged into arty.

### Coordinate spaces (the most common pitfall)

| file kind | space |
|---|---|
| dataset labels (`labels_simple`, `labels_detailed`, `label_detail_id`, `labels_delete`) | **native image pixels** |
| all prediction files, and `gt_p04v2` | **0.30 m grid = native px × `scale_to_target`** |

To draw predictions on the native .tif, divide the coordinates by the scene's `scale_to_target`. For example, a 0.79 m Pléiades scene has scale 2.633.

### Ground-truth files (native px)

| file | format |
|---|---|
| `labels_simple/<scene>.txt` | `x1 y1 x2 y2 x3 y3 x4 y4 class difficult`. This is the **official simple mapping**: P_Camo = Yes objects and camo parents are `camo`. |
| `labels_detailed/<scene>.txt` | 25 columns: coords, `det_class fine Normal Snow Dark Quality Functionality Position P_Camo P_Cut Remarks User Is_Empty SOC Camo_Instance Background_Matching difficult` |
| `label_detail_id/<scene>.txt` | the same 25 columns + **instance ID** (column 26, e.g. `18r01` = image 18, radar, number 01). The image number is base 36: `01`–`17` Test1, `18`–`28` Test2. |
| `labels_delete/<scene>.txt` | polygon + `Delete`. Predictions whose centroid falls inside are ignored by every scorer. |
| `class_mapping.json` (dataset root) | fine label → canonical class |

Rows of `labels_simple` and `label_detail_id` are in the same order (zip them).

Instance table for analysis: `/sfs/stage2/artifacts/EXP-2026-10-12-01-attr-pattern-merge/instances.tsv`. It has 5,308 objects × 57 columns:
- all attributes;
- the m0001 deploy match;
- the 8-class score vector;
- the best c05 score per class on each object.

## 3. Prediction file formats (0.30 m grid)

| folder name | columns | content |
|---|---|---|
| `c05` (`predictions_final_c05`, `preds/c05`) | `x1 y1 … x4 y4 class score` | every box with score ≥ 0.05. TTA 1024/1536 × flips, ensemble noisy-or. **This is what all scorers read.** |
| `deploy` (`predictions_deploy`, `preds/deploy`) | same | after the deployment post-processing (below) |
| `predictions_final_vectors` (m0001 only) | `coords class maxscore s_arty s_camo s_logistic s_missile s_radar s_smallvehicle s_tank s_vehicle` | the full 8-class score vector per box |
| `best_per_class` (EXP-2026-10-26-02) | `coords class score` | class c boxes from the best model for c, at score ≥ that class's best-F1 confidence |
| `gt_p04v2` (EXP-2026-10-26-02) | `coords class_p04v2 instance_id official_class` | GT moved to the 0.30 m grid with the P04 v2 class |

**Deployment post-processing** (m0001, decision D-20260902-01):
- per-class thresholds: radar 0.39, missile 0.85, arty 0.86, logistic 0.77, camo 0.42, smallvehicle 0.24, tank 0.54, vehicle 0.84;
- cross-class dedup at IoU 0.4, with priority radar > missile > arty > logistic;
- delete regions removed;
- script `code/engine_v1/apply_priority.py`.

## 4. Where every model's predictions are (21 prediction sets)

- **m0001** (production: YOLOv8m-OBB + YOLO11l-OBB ensemble)
  - Predictions: `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/` → `predictions_final_c05/`, `predictions_deploy/`, `predictions_final_vectors/`
  - Weights: `/sfs/stage2/models/BEST` → `m0001_sep_w1_ensemble/{prod_y8m_best.pt, prod_y11l_best.pt}`, plus its MODEL_CARD.md
- **The 20 runs from 2026-10-14/15:** `/sfs/stage2/artifacts/<EXP-ID>/preds/{c05,deploy}/labels/`, with weights in `runs/*/weights/best.pt`.
  - Each run's reports are in `reports/P01_detailed.md` and `rules.json`.
  - The notebook record is `experiment_obs/2026/2026-10/week-2026-10-11/experiments/<EXP-ID>/record_ai.md`.
  - EXP-2026-10-14-10 is m0001 re-run through the same pipeline (identical to m0001), so it is left out of the comparisons.

Each model scored as a whole (best-F1 confidence per class, IoU 0.3), MICRO F1 / MACRO F1:

| prediction set (EXP-2026-…) | P04 v2 MICRO | P04 v2 MACRO | official P01 MICRO | official P01 MACRO |
|---|--:|--:|--:|--:|
| 10-14-14 ens D-frozen y8m + y11l | **63.4** | **54.3** | **59.9** | 48.5 |
| m0001 ensemble (y8m + y11l) | 63.0 | 54.0 | 59.7 | **49.0** |
| 10-14-06 ens B y8m-ft + y11l | 62.8 | 53.3 | 59.4 | 48.3 |
| 10-14-16 ens D (y8m-D + y11l-D) | 62.6 | 53.6 | 59.3 | 48.5 |
| 10-14-08 ens D y8m-ft + y11l | 62.5 | 53.7 | 59.2 | 48.6 |
| 10-14-07 ens C y8m-ft + y11l | 62.4 | 53.2 | 59.2 | 48.5 |
| 10-14-09 ens control y8m-ft + y11l | 62.3 | 52.9 | 59.0 | 48.1 |
| 10-14-15 ens R y8m-ft + y11l | 62.2 | 52.9 | 58.9 | 48.2 |
| 10-15-01 ens y11l original + y11l-D | 61.9 | 51.9 | 58.4 | 46.7 |
| 10-14-17 y11l original alone | 61.9 | 52.0 | 58.4 | 47.0 |
| 10-14-13 y11l fine-tuned D | 61.7 | 51.8 | 58.4 | 46.3 |
| 10-14-18 y11l control fine-tune | 61.6 | 51.3 | 58.6 | 46.5 |
| 10-14-11 y8m D frozen backbone | 61.4 | 52.2 | 58.1 | 46.7 |
| 10-15-02 y11l-D low LR | 61.3 | 51.6 | 57.6 | 46.3 |
| 10-14-05 y8m original alone | 60.9 | 51.5 | 57.4 | 46.8 |
| 10-14-19 y11l DR | 60.8 | 51.0 | 57.6 | 46.3 |
| 10-14-03 y8m D | 60.8 | 51.5 | 57.2 | 46.4 |
| 10-14-01 y8m B | 60.6 | 51.6 | 57.5 | 46.4 |
| 10-14-12 y8m R | 60.6 | 51.0 | 57.6 | 46.1 |
| 10-14-04 y8m control | 60.6 | 50.7 | 57.1 | 45.8 |
| 10-14-02 y8m C | 60.5 | 51.1 | 57.2 | 46.2 |

Variant labels used for training (`code/engine_v1/build_variant_labels.py`):
- **A** unchanged.
- **B** non-launcher missiles → vehicle.
- **C** empty launchers → vehicle.
- **D** = B + C.
- **R** stowed radar → vehicle.

## 5. The best-per-class set (P04 v2, test_80, IoU 0.3, upper bound)

| class | GT | model used | conf ≥ | F1 | P | R | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 162 | EXP-2026-10-14-14 | 0.90 | 39.7 | 42.1 | 37.7 | 61 / 84 / 101 |
| camo | 548 | EXP-2026-10-14-14 | 0.80 | 50.0 | 50.8 | 49.3 | 270 / 262 / 278 |
| logistic | 241 | EXP-2026-10-14-13 | 0.60 | 28.6 | 26.7 | 30.7 | 74 / 203 / 167 |
| missile | 197 | EXP-2026-10-14-14 | 0.90 | 64.2 | 66.7 | 61.9 | 122 / 61 / 75 |
| radar | 190 | EXP-2026-10-14-17 | 0.60 | 58.8 | 71.4 | 50.0 | 95 / 38 / 95 |
| smallvehicle | 822 | EXP-2026-10-14-13 | 0.50 | 55.2 | 62.5 | 49.4 | 406 / 244 / 416 |
| tank | 744 | m0001 | 0.90 | 70.3 | 70.2 | 70.4 | 524 / 222 / 220 |
| vehicle | 2196 | m0001 | 0.90 | 72.9 | 68.7 | 77.6 | 1705 / 778 / 491 |

- **Totals:** MICRO P 63.3 / R 63.9 / **F1 63.6**, **MACRO F1 55.0** (TP 3,257, FP 1,892, GT 5,100). The rebuilt folder reproduces these exactly ([check](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-02-best-pred-files-guide/best_per_class_check_ai.json)).
- **m0001 deploy set under the same GT:** P 56.4 / R 63.1 / F1 59.6, MACRO 51.5.
- **Why the two sets differ:** the deploy set uses the lower deploy thresholds plus cross-class dedup, while the best set uses tuned per-class confidences.
- **The best set is not one deployable model.** It mixes 4 models. Classes are scored independently, so overlapping boxes of different classes are allowed.

## 6. Class mappings (protocols)

| protocol | what | status |
|---|---|---|
| P01 official | simple mapping (`labels_simple`). Camouflaged objects (P_Camo = Yes or camo parent) → `camo`; one-to-one. | official |
| P02 | camo credit (camouflaged object correct as camo or its hardware class) | history |
| P03 | several accepted classes per object (missile/radar rows) | retired for reporting, D-20261023-01 |
| **P04 v2** | one class per object: P01 + 15 attribute rules | **current draft**, D-20261023-01 + D-20261026-01 |

P04 v2 rules (`/sfs/stage2/artifacts/EXP-2026-10-26-01-pcamo-missile-radar-mapping/config/rules.json`; first matching rule applies):
- **→ missile:** camouflaged HQ9, S300, HQ12, MissileSystem.
- **→ vehicle:**
  - MLRS (camouflaged or not);
  - uncamouflaged HQ9;
  - uncamouflaged MissileSystem;
  - camouflaged HQ16;
  - EWS radar;
  - camo-parent LTV;
  - ABLV.
- **→ tank:** SPATracked, camouflaged tanks.
- **→ radar:** camouflaged radar (P_Camo = Yes).

Training labels for every model above use the labels_detailed class. P_Camo missiles and radars are trained as missile / radar; only camo-parent objects are trained as camo.

## 7. Scoring scripts (env `/sfs/envs/yolo_models/bin/python`: shapely, rasterio, ultralytics)

| script | does |
|---|---|
| `/sfs/stage2/code/engine_v1/report_from_preds.py --preds <c05 dir> --out <md>` | official P01 report from saved predictions. Per-class greedy matching, confidence 0.05–0.90, IoU 0.1/0.3/0.5. |
| `/sfs/stage2/code/engine_v1/apply_priority.py` | c05 → deploy (thresholds + dedup) |
| `/sfs/stage2/code/engine_v1/score_rules.py` | P03 rows (history) |
| `artifacts/EXP-2026-10-23-02-…/code/score_remap.py identity\|remap [--drop Rxx]` | P01 logic with a rules.json remap; scores all 21 runs in seconds |
| `artifacts/EXP-2026-10-26-01-…/code/compare_mapping.py` | the 15 camouflaged missile/radar mapping options |
| `artifacts/EXP-2026-10-26-02-…/code/build_best.py` | rebuilds the best_per_class folder, gt_p04v2 and per_image.tsv |

Matching convention everywhere:
- boxes are sorted by score;
- each box takes the best-IoU unmatched GT of the same class (IoU ≥ 0.3);
- one box ↔ one object;
- predictions in delete regions are dropped.

## 8. Findings so far (test_80, IoU 0.3)

1. **Best numbers so far** (best per class under P04 v2, upper bound): MICRO F1 **63.6**, MACRO **55.0**, R@P≥0.30 79.3 / 65.5. The same picks under official P01 give 60.2 / 49.6.
2. **Best single prediction set:** EXP-2026-10-14-14 (y8m D-frozen + y11l) at 63.4 / 54.3, just ahead of m0001 at 63.0 / 54.0. m0001 is still the production model and the best under official P01 MACRO.
3. **Camouflaged missiles and radars belong in their own class** (EXP-2026-10-26-01).
   - Under camo, missile F1 is 36.4; under missile it is 59.4–62.1. Camo also does better without them (46.5 → 49.5).
   - Camouflaged radar → radar: radar F1 +4.6 to +6.2, camo −1.2.
   - Reason: every model was trained with them as their own class.
4. **MLRS behaves like a truck.** 19 of 20 camouflaged MLRS and most uncamouflaged MLRS are called vehicle. Scoring MLRS as vehicle adds recall.
5. **Relabel-and-fine-tune variants (B / C / D / R) did not beat the originals overall.**
   - y11l-D has the most precise missile output (P 97.8 at R 50.8, P03 history).
   - Radar does best from the original y11l.
   - See the [week review](../2026/2026-10/week-2026-10-11/review_ai.md).
6. **Weakest classes:** logistic (F1 28.6; the model calls most logistic trucks "vehicle") and arty (39.7).
7. **Radar misses** (deep dive EXP-2026-10-12-01):
   - most are detection gaps;
   - rare and older radar types are almost never found;
   - 66 of 112 wrong radar boxes sit on empty ground.

**Caveats**
- All picks (rules, model per class, confidence) were made on test_80 itself, so the numbers are optimistic.
- A held-out split is needed before freezing anything.
- The 26 new 15_10 scenes are not in test_80 and have not been scored.

## 9. Related records

- **How the best combination is built and re-run:** [best_combination_ai.md](best_combination_ai.md) (`code/engine_v1/combine_best.py`)

- [EXP-2026-10-23-01 best per class](../2026/2026-10/week-2026-10-18/experiments/EXP-2026-10-23-01-best-per-class-oracle/record_ai.md)
- [EXP-2026-10-23-02 single-class remap (P04)](../2026/2026-10/week-2026-10-18/experiments/EXP-2026-10-23-02-single-class-attr-remap/record_ai.md)
- [EXP-2026-10-26-01 camouflaged missile/radar mapping](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-01-pcamo-missile-radar-mapping/record_ai.md)
- [decisions_ai.md](decisions_ai.md) · [week 2026-10-11 review](../2026/2026-10/week-2026-10-11/review_ai.md)
