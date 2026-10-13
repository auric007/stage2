<!-- stage2 experiment card -->
# e0002 · sensor-exp-selection

| field | value |
|---|---|
| id | e0002 |
| date | 2026-09-24 (folder week `2026-09/w4`) |
| type | analysis |
| status | done |
| protocol | - |
| headline | SELECTED_40 covers 56 % of the random->full test-coverage gap |
| imported from | `/sfs/weekly_exp/sep_w1/sensor_exp` (copied into stage2 on 2026-10-13; original folder may be deleted) |

**What:** Pick 40 train images (dense + sparse rare-class) for same-area re-collection from another sensor; 25-sensor list.

**Note:** Waiting on analyst re-collection (arm B) - not trained yet.

---
*Original documentation follows (paths repointed to stage2 on import; logs and `args.yaml` keep their historical paths).*

# sensor_exp — does the training sensor matter?

**Question:** we keep the scenes, dates, objects and test set the same and change only the sensor
the training images came from. Does test recall change?

- **A** = train on 40 selected Train images as they are now (mostly WV02)
- **B** = the same 40 areas re-collected from a different sensor (same date or within ±7 days), re-annotated
- (optional **C** = A + B, 80 images)
- Test on the unchanged 80-image test set (Test1 + Test2).

## Files
| file | what |
|---|---|
| `SELECTED_40.txt` | **the 40 image stems** (one per line) to give to the analysts |
| `SELECTED_40.tsv` | same + bucket, per-class counts, current sensor, GSD, datetime, lat/lon, gains, candidate alt sensors |
| `RESERVE_10.txt` / `.tsv` | ranked backups if an area has no same-date collection from another sensor (3 sparse / 4 medium / 3 dense) |
| `SENSOR_LIST.txt` | 22 optical + 3 SAR candidate platforms, priorities, delivery rules |
| `embed_objects.py` | object embeddings (FINAL prod_y8m, neck P3+P4 ROI-mean) for all 21,673 train / 5,091 test objects → `obj_emb.npz` |
| `select_images.py` | the selection (below) → the lists above + `selection_state.json` |

## How the 40 were chosen
1. **Influence on test = class-conditional facility location.** Each test object is embedded with the
   final model's features. An image's similarity to a test object is the best cosine to any object of
   the **same class** in that image (features centred first, because raw post-SiLU cosines saturate at
   ~0.93). Greedy maximises Σ_test w · max-similarity over the chosen set. The returns diminish, so
   once a kind of test object is covered, further copies of it add nothing.
2. **Rare classes weigh as much as common ones:** w = 0.75/n_test[class] + 0.25/n_test_total. So
   covering the 206 test radars matters as much as covering the 2,053 test vehicles.
3. **Not only crowded scenes:** at least 12 sparse (≤30 objects), at most 10 dense (>150 objects),
   5–400 objects per image. All 12 sparse picks contain rare classes (radar/missile/arty/logistic).
4. Max 2 images per site (2 km), GSD ≤ 0.80, and at least 4 alternative sensors in orbit on the acquisition date.

## Result of the selection
- Mix: 10 dense / 18 medium / 12 sparse. Sensors: 37 WV02, 2 PHR-1A, 1 PHR-1B. Dates 2017–2026.
- 4,123 objects: vehicle 1,815 · smallvehicle 722 · camo 410 · logistic 360 · tank 282 · missile 228 ·
  arty 217 · radar 89. Every class is present (radar in 26 images, missile in 23).
- Test coverage objective: random 40 = 4.27, **selected 40 = 4.58**, all 316 = 4.82. The 40 recover
  56% of the gap between a random 40 and the full training set.

## Experiment protocol
- Same pipeline for A and B: GSD-normalise to 0.30 m, 1024 tiles with 20% overlap, RGB = bands 5,3,2
  (or the new sensor's equivalent R,G,B), single YOLOv8m-OBB, same hyperparameters, **no GE booster**
  (it would hide the sensor effect), 3 seeds per arm.
- B needs **new labels**: the re-collected image doesn't line up pixel-for-pixel with the old one, so
  analysts re-annotate, or old labels are moved via georeferencing and every box is reviewed.
- Score with `engine/report_from_preds.py`: recall @ P ≥ 0.30, IoU 0.3/0.5, per class, micro/macro,
  radar/missile top-k, and results **split by test sensor** (WV02 70 / Pléiades 7 / GeoEye-1 3).
  Caveat: the test set is 88% WV02, so A has a same-sensor advantage, and the per-sensor split shows it.

Re-run: `cd /sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train && /sfs/envs/yolo_models/bin/python3 sensor_exp/embed_objects.py && /sfs/envs/yolo_models/bin/python3 sensor_exp/select_images.py`
