<!-- stage2 experiment card -->
# e0003 · topk-disagree-export

| field | value |
|---|---|
| id | e0003 |
| date | 2026-09-24 (folder week `2026-09/w4`) |
| type | analysis |
| status | done |
| protocol | P01 |
| headline | 879 boxes (tank 209, camo 137, vehicle 137, logistic 104, missile 102, ...) |
| imported from | `/sfs/weekly_exp/sep_w4/n0001_topk_disagree` (copied into stage2 on 2026-10-13; original folder may be deleted) |

**What:** Export test boxes where top-1 != GT class but GT is rank 2/3 (IoU 0.3), n0005 format, native px.



---
*Original documentation follows (paths repointed to stage2 on import; logs and `args.yaml` keep their historical paths).*

# sep_w4 / n0001 — top-k disagreement boxes (possible mislabels / close-class confusions)

These are test objects where the **FINAL sep_w1 ensemble** (YOLOv8m-OBB + YOLO11l-OBB, TTA, noisy-or) finds
the object, but its **top-1 class disagrees with the GT class**, while the **GT class is still the box's
2nd or 3rd score**. They are candidates for label review: either the label is wrong, or the model is
confused between two close classes.

Same layout as `/sfs/data/up42_platform_data_27_08_with_soc/n0005_class_agnostic/`.

## Contents
```
predictions/<scene>.txt   x0 y0 x1 y1 x2 y2 x3 y3  <pred_top1_class>  <top1_score>
ground_truth/<scene>.txt  x0 y0 x1 y1 x2 y2 x3 y3  <gt_class>  0
                          (line i of predictions ↔ line i of ground_truth = the same object)
disagreements.csv         one row per box: gt_class, pred_top1, gt_rank (2|3), IoU, top-1/2/3 classes+scores,
                          GT-class score, both polygons (native px)
summary.csv               per scene: split, native W/H, #disagree, #rank-2, #rank-3, usable
stats.json                counts per class + confusion pairs
viz/                      overlay check (green = GT, red = prediction, label gt>pred)
build.py                  re-generates everything
```
All 80 test scenes have a file (74 contain at least one box; the rest are empty files).

## Coordinates
**Native TIFF pixels**, the same space as `<split>/labels_detailed/` and n0005. They overlay
`<split>/images/<scene>.tif` directly. The ensemble predicts in the 0.30 m working space, so boxes
were mapped with `÷ scale_to_target` (not `÷ 2×scale` as in n0005, which ran on 2×-SR renders).
Check: all 879 GT boxes coincide with n0005's native GT, and 0 predicted boxes fall outside the image.

## Selection rule
- Source: `sep_w1/predictions_final_vectors/` (per-box 8-class score vectors, ≥0.05 floor).
- Each GT object is matched to its best-overlapping box with **IoU ≥ 0.3** and the highest max-score, the same
  logic as `sep_w1/FINAL/FINAL_TOPK.md`. Boxes inside `labels_delete` ignore regions are skipped.
- Kept only if the GT class has **rank 2 or 3** in that box. Objects whose GT class is outside the
  top 3, and objects with no box at all, are **not** included.
- GT class = the `true` mapping (col 8 of labels_detailed).

## Counts (IoU 0.3) — total **879**; these reproduce FINAL_TOPK.md (top-3 − top-1)
| GT class | boxes | GT rank 2 | GT rank 3 |
|---|--:|--:|--:|
| arty | 34 | 18 | 16 |
| camo | 137 | 59 | 78 |
| logistic | 104 | 82 | 22 |
| missile | 102 | 72 | 30 |
| radar | 61 | 40 | 21 |
| smallvehicle | 95 | 60 | 35 |
| tank | 209 | 128 | 81 |
| vehicle | 137 | 119 | 18 |

## Most common confusions (GT → predicted top-1)
| pair | n |
|---|--:|
| tank->vehicle | 150 |
| logistic->vehicle | 93 |
| smallvehicle->vehicle | 86 |
| missile->vehicle | 82 |
| camo->tank | 77 |
| camo->vehicle | 51 |
| vehicle->smallvehicle | 48 |
| tank->camo | 43 |
| radar->vehicle | 38 |
| vehicle->logistic | 34 |
| arty->tank | 25 |
| vehicle->tank | 23 |

## ⚠️ Unusable scene
`23JAN27044234-P3DS-200012483678_01_P001` (known-bad thumbnail, see n0005 README) → `usable = NO`
in summary.csv. It has 0 disagreement boxes.

Built 2026-09-24.
