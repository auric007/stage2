# test_pred_05 — every prediction with score ≥ 0.05, 80 test images

Model: the FINAL 8-class ensemble from `/sfs/weekly_exp/sep_w1` (YOLOv8m-OBB + YOLO11l-OBB, TTA scales
1024/1536 × flips none/h, noisy-or fusion, poly-NMS 0.4). Source: `sep_w1/predictions_final_c05/labels/`,
converted to JSON, nothing removed. Built 2026-10-05.

Test set: 80 images = Test1 (43) + Test2 (37) of `/sfs/up42_platform_data_27_08_with_soc`
(same 80 as `/sfs/end_to_end/oct_w1/images.txt`).

## Files
| file | what |
|---|---|
| `predictions_all.json` | all 80 images in one file |
| `per_image/<scene>.json` | the same records, one file per image |

## Content
- **23,041 boxes**: vehicle 9,454 · smallvehicle 4,606 · camo 2,365 · logistic 2,017 · tank 1,943 ·
  arty 987 · missile 981 · radar 688.
- **Every box with score ≥ 0.05** (min = 0.05). No per-class thresholds and **no cross-class de-duplication**:
  the same object can appear under several classes, each with its own score. Apply your own thresholds.
- Boxes per image sorted by score (descending). `id` = rank within the image.

```json
{"scene": "...", "split": "Test1", "image": "<scene>.tif", "native_size_wh": [W, H],
 "gsd_m_per_px": 0.66, "scale_to_target": 2.2, "n_boxes": 312, "class_counts": {...},
 "boxes": [{"id": 0, "class": "radar", "score": 0.9995,
            "polygon_native":  [[x,y],[x,y],[x,y],[x,y]],
            "polygon_norm030": [[x,y],[x,y],[x,y],[x,y]]}]}
```

## Coordinates
- `polygon_native`: **native TIFF pixels**, so it overlays `<split>/images/<scene>.tif` and matches `labels_detailed`
  and stage 1's ground truth.
- `polygon_norm030`: the model's 0.30 m/px working grid. `norm030 = native × scale_to_target`, where
  `scale_to_target = gsd / 0.30`.
- 4 corners per box, clockwise. Origin top-left, x right, y down.

## Notes
- During the lab's inference, **boxes whose centre fell inside a ground-truth ignore region
  (`labels_delete`) were dropped**. A live run on new imagery would return those boxes too.
- Check against GT at IoU ≥ 0.3 (GT includes aerial/radome/concealment objects, which the model doesn't detect):

  | images | GT objects | a box on it | a box of the correct class |
  |---|--:|--:|--:|
  | **all 80** | 5,308 | 4,747 (89.4 %) | 4,192 (79.0 %) |
  | 79 (excl. 23JAN27044234) | 5,301 | 4,747 (89.5 %) | 4,192 (79.1 %) |

  `23JAN27044234-P3DS-200012483678_01_P001` is included in the all-80 numbers. It has 7 GT objects, 0 matched, and 77 predicted boxes.
  Its TIFF is a 497×549 thumbnail while its GT coordinates reach ~1,350 px (labels drawn on the full-res
  scene), so the predictions and GT are in different scales and can't match. Its predictions are included as-is.
