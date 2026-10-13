<!-- stage2 experiment card -->
# e0001 · final-ensemble-train

| field | value |
|---|---|
| id | e0001 |
| date | 2026-09-01 (folder week `2026-09/w1`) |
| type | train |
| status | done |
| protocol | P01 |
| headline | recall-first IoU0.3 MICRO 74.4 / MACRO 58.0; deploy P 53.6 / R 60.0 |
| imported from | `/sfs/weekly_exp/sep_w1 (minus sensor_exp)` (copied into stage2 on 2026-10-13; original folder may be deleted) |

**What:** Final 8-class ensemble: YOLOv8m-OBB (68 ep) + YOLO11l-OBB (78 ep), GE booster, TTA 1024/1536 x flips, noisy-or. Deploy post-processing, top-k analysis, inference package, 5-scene bundle.

**Note:** Promoted as model m0001 (BEST). `predictions_final/` is EMPTY (infer_test GT bug) - use predictions_final_c05 / predictions_deploy.

---
*Original documentation follows (paths repointed to stage2 on import; logs and `args.yaml` keep their historical paths).*

# sep_w1 — Final all-class detection model (deployable)

Production 8-class oriented-bbox detector, **ensemble (YOLOv8m-OBB + YOLO11l-OBB) + TTA + GE booster**
— the best recipe from july_w4, trained on the complete 27_08 data with **no SOC segregation**.
Built 2026-09-01 (moved here from the aug_w4_2 lab, which holds the earlier SOC-attribute study).

## Result (recall-first, P ≥ 0.30, 80-image held-out test — leakage-free)
| | MICRO | MACRO |
|---|--:|--:|
| **IoU 0.3** | **74.4 %** | **58.0 %** |
| IoU 0.5 | 72.4 % | 55.6 % |

Per-class recall @P≥0.30, IoU 0.3: vehicle 91.6 · tank 91.1 · smallvehicle 66.2 · radar 57.4 ·
camo 56.8 · missile 41.4 · arty 37.4 · logistic 22.4. Full breakdown in `FINAL/FINAL_ENSEMBLE.md`.

## Recipe
- 8 classes: arty, camo, logistic, missile, radar, smallvehicle, tank, vehicle (radar/missile separate).
- GSD-normalize every image → 0.30 m, tile 1024/20 %, 8-band TIFF → RGB=(bands 5,3,2).
- Train = 27_08 Train (10,758 tiles) + GE gepack booster (918 canvases / 35,682 objs, rare-class rich).
- Members: `prod_y8m` (68 ep), `prod_y11l` (78 ep), 100 ep / patience 30, scale-aug 0.3, AdamW.
- Inference: **TTA scales {1024,1536} × flips {none,h} + noisy-or + NMS 0.4**.

## Files
| path | what |
|---|---|
| `FINAL/prod_y8m_best.pt`, `FINAL/prod_y11l_best.pt` | **the two ensemble weights — deploy these** |
| `FINAL/FINAL_ENSEMBLE.md` | headline + per-class + inference command |
| `FINAL/FINAL_REPORT_DETAILED.md` | per-class P/R sweep, IoU 0.1/0.3/0.5, agnostic + detectability |
| `FINAL/FINAL_SCOREWISE.md` | TP/FP/FN + P/R/F1 at every confidence |
| `predictions_final/labels/` | ⚠ **all 80 files EMPTY**: infer_test derived its thresholds from the mis-loaded (all-zero) GT, so every class got thr 1.0 (see SUMMARY.txt). Do not use; take `predictions_final_c05` (all ≥0.05) or `predictions_deploy` (final thresholds + dedup) |
| `predictions_final_c05/labels/` | all boxes ≥ 0.05 (threshold yourself) |
| `predictions_final_vectors/labels/` | per-box **8-class score vectors** (18 col: coords+argmax+maxscore+s_arty…s_vehicle), ensemble multi-label |
| `FINAL/FINAL_TOPK.md` | **top-1/2/3** class analysis — is each GT's true class in its box's top-k? (IoU 0.3 & 0.5) |
| `runs/prod_y8m`, `runs/prod_y11l` | full training runs (weights, results.csv, args) |
| `engine/` | pipeline (common.py repointed to this lab) |
| `data/final/` | training dataset (image symlinks → aug_w4_2/data/tiles_norm) |
| **`infer_point_stage2/`** | **self-contained inference package** — weights + preprocessing + docs; anyone can run it on new PNGs (see its README.md). Zipped: `infer_point_stage2.zip` |
| `sample_bundle_5scenes/` (+`.zip`) | 5 sample test scenes: TIFF, renders, GT, all 3 pred sets, overlays, DOCS.md |
| `run_final.sh`, `run_final_resume.sh` | train + ensemble driver (provenance / re-run) |
| **`sensor_exp/`** | sensor-comparison experiment: `SELECTED_40.txt` (40 train images to re-collect from another sensor), `RESERVE_10`, `SENSOR_LIST.txt`, selection method in its README |

## Run inference
```bash
PY=/sfs/envs/yolo_models/bin/python3
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
cd /sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train
# on the existing 80-image test (reproduces predictions_final):
$PY engine/infer_test.py --runs prod_y8m_s0,prod_y11l_s0 \
    --scales 1024,1536 --flips none,h --nms 0.4 --conf 0.05 --fuse noisyor --out predictions_final
# score (use report_from_preds — NOT infer_test's headline, which mis-loads GT for this layout):
$PY engine/report_from_preds.py --preds predictions_final_c05/labels --out FINAL/FINAL_REPORT_DETAILED.md
```
For **new imagery**: put images+metadata in the dataset layout, rebuild `state/gsd_norm.json`, tile with
`engine/prep_soc.py`, then run the infer command above.

## Notes
- ⚠ `infer_test`'s own TEST_RESULTS headline reports **all-zero GT** for this tile layout (a known
  quirk) — always score with `report_from_preds`. `FINAL/FINAL_ENSEMBLE.md` was written from the
  correct `report_from_preds` numbers.
- Weakest classes: **logistic (22 %)** and **arty (37 %)** — precision-limited (many FP); next lever
  if needed.
- The **SOC-attribute study** (radar/missile Pixels/Site/Third_Party, A0–A3 ablation) lives in
  `/sfs/weekly_exp/aug_w4_2` — a separate line of work.
