<!-- stage2 experiment card -->
# e0005 · imagewise-deploy

| field | value |
|---|---|
| id | e0005 |
| date | 2026-10-08 (folder week `2026-10/w2`) |
| type | eval |
| status | done |
| protocol | P01 |
| headline | reproduces deploy totals exactly (TP 3059 / FP 2646 / GT 5100) |
| imported from | `/sfs/weekly_exp/oct_w2/n0001_imagewise_deploy` (copied into stage2 on 2026-10-13; original folder may be deleted) |

**What:** Per-image confusion matrix, per-class P/R and instance-ID tables for all 80 test images at deploy thresholds.



---
*Original documentation follows (paths repointed to stage2 on import; logs and `args.yaml` keep their historical paths).*

# oct_w2 / n0001 — image-wise analysis of the deployment predictions

Per-image confusion matrix, per-class precision/recall and an instance-by-instance result list for
all 80 test images, at the **deployment confidence thresholds**. Built 2026-10-08.

| file | what |
|---|---|
| [`SUMMARY.md`](SUMMARY.md) | totals for all 80 images + one row per image (worst F1 first) with links |
| `images/<II>_<scene>.md` | per image: headline P/R/F1, per-class table, confusion matrix, every GT instance (by ID), false-alarm list |
| `summary.json` | the same numbers, machine-readable |
| `build.py` | regenerates everything (`/sfs/envs/yolo_models/bin/python3 build.py`) |

`<II>` is the image number from your instance IDs (base 36: `01`–`17` Test1, `18`–`28` Test2), so
files sort in ID order and `09_…md` holds instances `09xNN`.

## Inputs
- **Predictions:** `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/predictions_deploy/labels/`. This is the sep_w1 FINAL ensemble (YOLOv8m-OBB +
  YOLO11l-OBB, TTA 1024/1536 × flips, noisy-or) after the deployment post-processing:
  - per-class thresholds: radar 0.39 · missile 0.85 · arty 0.86 · logistic 0.77 · camo 0.42 · smallvehicle 0.24 · tank 0.54 · vehicle 0.84;
  - cross-class de-duplication at IoU 0.4, with priority classes radar, missile, arty and logistic winning a shared region;
  - boxes whose centre is inside a `labels_delete` region are removed.
- **Ground truth:** `/sfs/data/up42_platform_data_27_08_with_soc/{Test1,Test2}`.
  - Class = `labels_simple`. Instance ID = column 26 of `label_detail_id` (same line order).
  - `tests_labels_id/` is byte-identical to `Test*/label_detail_id/`.

## Decisions (agreed 2026-10-08)
| question | decision |
|---|---|
| GT class | **simple mapping**: an object with P_Camo = Yes counts as `camo`; `spa` counts as `arty`. The instance table also shows the true class, so e.g. a camouflaged radar appears as GT `camo`, true class `radar` |
| classes the model never learned (aerial 172, concealment 19, radome 17) | **dropped completely**: not in any table. A box on one of them counts as a false alarm. Each image file lists how many were dropped |
| IoU gate | **0.3** |
| known-bad thumbnail `23JAN27044234` (image `1M`) | **included**, flagged. Its GT is off-scale, so it scores 0 |

## Matching rule
1. **Correct (TP).** Per class, boxes are taken in score order. Each takes the first not-yet-matched GT of the
   **same** class with IoU ≥ 0.3. This is exactly the rule of `sep_w1/engine/apply_priority.py`, so the
   80-image totals equal `predictions_deploy/STATS.md` (TP 3,059 · FP 2,646 · GT 5,100 · P 53.6 % · R 60.0 %).
2. **Wrong class.** For the confusion matrix, a leftover box is paired with the best leftover GT of any other class at IoU ≥ 0.3.
   That GT is counted as `wrong class` (an FN) and the box is an FP of its own class.
3. **Missed.** A GT with no box at all.
4. **False alarm.** A box on no scored object.

Precision = TP / boxes, recall = TP / GT, per image and per class within the image.

## Caveats
- 5 scored GT objects (all smallvehicle; 4 in `20NOV12043850`, 1 in `22MAY17025959`) sit inside delete regions,
  where boxes were removed, so they can only count as missed. They are left in to keep the totals equal to the deployment report.
- Boxes on the dropped classes count as false alarms (decision above). That mostly affects images with aircraft.
- Coordinates in the instance tables are object centres in **native TIFF px**.
- Each split's `metadata/` entries are symlinks to the pre-move path, which are broken. `build.py` reads the raw export
  `/sfs/data/up42_platform_data_27_08/metadata/` instead.
