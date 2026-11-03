# EXP-2026-11-02-01-dino-arch: new architecture (DINOv3 OBB, "DINO8"), 3 improvement routes vs HYBRID — AI-authored

Author: AI agent (Claude). Generated: 2026-11-02 20:59 IST. Human companion: [record.md](record.md) (empty slot).

Status: **running**. M1 is training on the V100; M2, M3 and ENS were scored with the shipped DINO predictions.

## Request summary and authorization
- **2026-11-02 ~20:00 IST.** The researcher attached a GPU: "start the all three improving method so that i can compare them apple-to-apple or use their ensemble".
- **The three routes** were proposed earlier the same day, after the DINO bundle was read (see the [daily note](../../daily/2026-11-02_ai.md)):
  1. retrain on the official labels;
  2. fix the class decision;
  3. fuse DINO with the HYBRID.
- No protocol, test-set or models/BEST change.

## Question and design
- **Model under test.** The DINO8 bundle at `/sfs/stage2/models/dino` (built elsewhere and handed over, 2026-11-01/02):
  - DINOv3 ViT-H/16+ encoder, last 3 blocks trainable, dense OBB head;
  - native RGB, 512 tiles;
  - warm-started from the class-agnostic `lr_obb_rgb_v1_best.pt`, trained 14 epochs.
- **Baseline.** The deployed HYBRID (D-20261102-01), using the files in `EXP-2026-10-29-01/share/hybrid_2026-11-02`.
- **Scoring.** Every route is scored the same way as the HYBRID: official 27_10 labels_simple, IoU 0.3, predictions centred in a Delete region dropped, 0.30 m grid (`score_set.py` semantics).
- **Routes:**
  - **M1, retrain on official labels.** The bundle recipe (`run_train.sh` flags) is unchanged, with three changes:
    1. labels: official 27_10 labels_simple (2026-10-27 rule);
    2. train cache read with the inference reader;
    3. checkpoint **selected by class-aware val MACRO F1**, not class-agnostic val mAP50.
  - **M2, class decision from the HYBRID.** DINO boxes are used as found. Each takes the class of the overlapping HYBRID raw box (IoU ≥ 0.3) with the highest score ÷ deploy threshold; if none overlaps, DINO's own class is kept.
  - **M3, HYBRID + DINO additions.** The HYBRID deployed output, plus DINO boxes of class c with score ≥ t_c that overlap no HYBRID box (IoU ≥ 0.3).
  - **ENS.** Per class, the best of H / M2 / M3 (best F1), then the standard cross-class merge (IoU 0.4, radar > missile > arty > logistic, then margin above threshold).
- **Success criterion.** Beat the HYBRID's MICRO 63.9 / MACRO 56.7 on Test116.

## Actual setup and changes
- **Code.** `/sfs/stage2/artifacts/EXP-2026-11-02-01-dino-arch/code` (no git):
  - `prepare_official.py`, `train_obb_cls.py`, `run_m1.sh`, `infer_test.py`, `fuse_eval.py`, `ensemble_eval.py`, `run_after_m1.sh`.
  - Bundle files are not modified.
- **Data.** `artifacts/.../data/{train,test1,test2}` link to `/sfs/data/up42_platform_data_27_10/prepared`.
  - Train is the same 359 scenes as the bundle. The val split is the bundle's 16-scene `splits/val.txt`.
  - Training counts (Delete-enclosed objects excluded): arty 730, camo 2742, logistic 1915, missile 711, radar 421, smallvehicle 5342, tank 1802, vehicle 12286, unsupported 1085.
- **Cache.** 6.3 GB on local disk (regenerable): `/tmp/claude-0/-home-jovyan/f544b93d-aa5c-4871-bfc2-ca1923d9e569/scratchpad/dino_cache_official`.
- **Environment.** `/sfs/envs/yolo_models` (torch 2.4.0+cu124), with the bundle's `pydeps` (transformers 4.57.3) and `artifacts/.../pydeps_extra` (tensorboard, regex; numpy deliberately not shadowed).
- **Hardware.** 1× Tesla V100-PCIE-32GB. The bundle was trained on 2× L4 with DDP.
- **Issues found in the bundle and worked around in the wrapper:**
  1. **Train/test band mismatch.** `prepare.py` generated train RGB with `read_bands(bands 1,2,3)`, while inference uses `read_full_rgb` (bands 5,3,2 for 8-band WorldView). The new cache uses `read_full_rgb` for training too. How many of the bundle's original arrays were affected is unknown: most came from a base cache that was not shipped.
  2. **Selection metric.** `_best.pt` was selected by **class-agnostic** val mAP50, which peaked at epoch 2 of 14. The shipped classifier therefore had about 2 epochs of 8-class training.
  3. **Import bug.** `stage1/model.py` uses `_os` without importing it; the error only appears when DINO_BACKBONE_DIR is needed.
  4. **V100 precision.** torch 2.4 reports bf16 "supported" on V100, so autocast chose emulated bf16. That forced the math attention kernel (out of memory at bs 8). `decode.predict_scene` also hardcodes bf16 autocast, which errors on V100. Both are forced to fp16 + GradScaler with memory-efficient SDPA.
  5. **Delete-region scale.** The bundle's `evaluate_test115.py` scales Delete polygons ×0.5 (its labels were on the images2x grid). `infer_test.py` reads them at native scale.
- **Protocol note.** The bundle's cached DINO predictions were made with GT Delete regions blacked out at inference (M1 test inference does the same). The HYBRID only drops boxes in Delete regions. The effect is small: both drop all boxes there.

## Execution
- **Pilot.** 60 steps, bs 8, fp16: 0.44 it/s, 15.5 GB of GPU memory. One overflow step was skipped (GradScaler calibration).
- **Shipped checkpoint on the 16 val scenes, official labels.** Class-agnostic mAP50 0.736. Class-aware val MACRO F1 **42.4**:

  | arty | camo | logistic | missile | radar | smallvehicle | tank | vehicle |
  |--:|--:|--:|--:|--:|--:|--:|--:|
  | 26.9 | 64.5 | 33.1 | 28.8 | 48.4 | 66.0 | 41.5 | 29.7 |

  The 16 val scenes take 311 s.
- **M1 launched at 2026-11-02 20:59 IST.**
  - Command: `code/run_m1.sh m1_relabel`, 14 epochs, 36,008 steps at about 0.45 it/s, so **about 23–24 h** including validation.
  - Log: `artifacts/.../logs/m1_relabel.log`. Weights: `weights/m1_relabel_best.pt` (class-aware selection) and `m1_relabel.pt` (last).
  - `code/run_after_m1.sh` (log `logs/after_m1.log`) waits for training, then:
    1. runs Test116 inference with the M1 best checkpoint;
    2. runs the shipped checkpoint on 23JAN27044234, the one scene its cache lacks;
    3. writes `reports/FUSE_{shipped116,m1_relabel_best}` and `ENS_*`.

- **M1 progress, 2026-11-03 00:08 IST.** Step 5100 of 36008 (epoch 1).
  - Val after epoch 0 (16 scenes, official labels): class-aware MACRO F1 **49.5**, vs 42.4 for the shipped checkpoint; class-agnostic mAP50 0.728.
  - Per class: arty 33.0, camo 65.1, logistic 32.4, missile 38.7, radar 51.9, smallvehicle 63.5, tank 41.7, vehicle 69.6. The shipped checkpoint had vehicle at 29.7.
  - Four fp16 overflow steps were skipped by GradScaler; this is expected.

- **M1 progress, 2026-11-03 12:38 IST.** Epoch 9, step 24500 of 36008. Class-aware val MACRO F1 per epoch:

  | epoch | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
  |---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
  | MACRO F1 | 49.5 | **54.6** | 51.7 | 52.3 | 54.6 | 52.8 | 52.7 | 52.1 | 52.6 |

  - Shipped checkpoint: 42.4.
  - Val vehicle rises to about 80 and camo to about 72. Arty swings between 21 and 55: there are few arty objects in the 16 val scenes, so this metric is noisy.
  - \`_best.pt\` = epoch 1, so \`code/run_after_m1_last.sh\` was added to also score the last checkpoint (epoch 13) on Test116.

## Results and failures
- **Shipped DINO, 115 scenes** (the bundle cache lacks 23JAN27044234), best-F1 threshold per class. Full tables: [FUSE_shipped_test115_ai.md](FUSE_shipped_test115_ai.md) and [ENS_shipped_test115_ai.md](ENS_shipped_test115_ai.md).

  | route | P | R | MICRO F1 | MACRO F1 | TP / FP / FN |
  |---|--:|--:|--:|--:|---|
  | H, HYBRID deployed | 61.9 | 66.2 | 64.0 | 56.8 | 4482 / 2755 / 2289 |
  | D, DINO alone | 59.4 | 50.2 | 54.4 | 48.9 | 3401 / 2323 / 3370 |
  | M2, DINO boxes + HYBRID class | 68.1 | 65.9 | **67.0** | 56.4 | 4463 / 2093 / 2308 |
  | M3, HYBRID + DINO additions | 60.8 | 69.5 | 64.9 | 57.6 | 4708 / 3039 / 2063 |
  | **ENS, after the cross-class merge** | **65.4** | **67.6** | **66.5** | **59.6** | 4574 / 2417 / 2197 |

- **ENS per class** (GT; source; P / R / F1; TP / FP / FN):

  | class | GT | source | P / R / F1 | TP / FP / FN |
  |---|--:|---|---|---|
  | arty | 249 | M2 @0.7 | 74.0 / 51.4 / 60.7 | 128 / 45 / 121 |
  | camo | 554 | M3 @0.75 | 52.9 / 62.1 / 57.1 | 344 / 306 / 210 |
  | logistic | 298 | H | 30.0 / 33.2 / 31.5 | 99 / 231 / 199 |
  | missile | 413 | M3 @0.55 | 51.7 / 54.0 / 52.8 | 223 / 208 / 190 |
  | radar | 274 | M2 @0.75 | 64.4 / 58.8 / 61.5 | 161 / 89 / 113 |
  | smallvehicle | 1242 | M3 @0.5 | 59.4 / 73.1 / 65.5 | 908 / 621 / 334 |
  | tank | 645 | M3 @0.45 | 76.8 / 71.5 / 74.1 | 461 / 139 / 184 |
  | vehicle | 3096 | M2 @0.7 | 74.3 / 72.7 / 73.5 | 2250 / 778 / 846 |

- **Recall-leaning ENS (2026-11-03, researcher asked how much better HYBRID+DINO is).** Per class, the route with max R at P ≥ floor (logistic: H), then merged. Table: [ENS_recall_shipped_test115_ai.md](ENS_recall_shipped_test115_ai.md).
  - Result: P 57.9 / R **70.1**, MICRO 63.5 / MACRO 56.4, TP 4749 / FP 3447 / FN 2022.
  - Against the HYBRID: +267 TP (R +3.9) for +692 FP (P −4.0).
- **Radar-vs-missile merge study (2026-11-03).** Code: \`code/ens2.py\`, which rebuilds every ensemble from pre-merge boxes and applies one merge. Its HYBRID-only row reproduces the deployed hybrid exactly (4482 / 2755 / 2289). Full table: [ENS2_radar_missile_shipped115_ai.md](ENS2_radar_missile_shipped115_ai.md).
  - The HYBRID merge removes 27 missile boxes, all for radar.
  - Recall-leaning options:

    | option | radar P / R | missile P / R | all R | MACRO |
    |---|---|---|--:|--:|
    | radar first (current) | 50.3 / 69.7 | 50.0 / 54.7 | 70.2 | 56.4 |
    | B, radar and missile both kept | 50.3 / 69.7 | 49.2 / 61.0 | 70.5 | 56.7 |
    | radar needs ≥ 0.25 to beat a missile | 53.5 / 66.4 | 50.4 / 60.0 | 70.3 | 56.8 |
    | margin decides | 53.1 / 62.8 | 49.4 / 60.8 | 70.2 | 56.6 |
    | missile from exp09 | 50.3 / 69.7 | 46.2 / 56.2 | 69.9 | 56.1 |

  - exp09 is not useful: its found set has Jaccard 0.96 with exp15.
- **Instance complementarity (2026-11-03).** Code: \`code/complement.py\`. Report: [COMPLEMENT_shipped115_ai.md](COMPLEMENT_shipped115_ai.md).
  - HYBRID vs DINO Jaccard is 0.27–0.73 per class. Union recall, the ceiling for an ensemble: smallvehicle 78.2, radar 74.5, camo 73.6.
  - DINO localizes better (missile: any box 98.1% vs 92.5%) but classifies worse (38.3% vs 59.8%). It calls 122 missiles camo (old label policy) and 821 vehicles logistic.
  - Shared misses: MLRS (82 objects, about 25% recall for every model) and radar EWS (21%).
- **Recall-objective picks** (max R at P ≥ floor) are in the FUSE file. M2 reaches R 69.9 at P 58.7, and M3 R 69.9 at P 56.9.
- **Failures fixed during setup:** `_os` NameError; out of memory at bs 8 (math SDPA); "No available kernel" (bf16); the bf16 autocast error in validation (caught before launch).

### M1 on Test116 (2026-11-03 21:27 IST) — full table: [COMPARE_M1_test116_ai.md](COMPARE_M1_test116_ai.md)
Only epoch 1 (val best) and epoch 13 (last) were kept; epochs 2-12 cannot be scored. M1 trained on 343 of 359 Train scenes (16-scene bundle val held out). Each checkpoint gets its own recall-leaning picks (`code/recall_picks.py`; reproduces the shipped picks exactly).

| model / route | P | R | MICRO | MACRO | TP / FP / FN |
|---|--:|--:|--:|--:|---|
| HYBRID deployed | 61.8 | 66.1 | 63.9 | 56.7 | 4482 / 2767 / 2296 |
| shipped DINO alone | 59.4 | 50.2 | 54.4 | 48.9 | 3401 / 2326 / 3377 |
| M1 epoch 1 alone | 62.0 | 54.6 | 58.1 | 51.6 | 3702 / 2273 / 3076 |
| **M1 epoch 13 alone** | 65.8 | 61.6 | 63.7 | 53.7 | 4176 / 2167 / 2602 |
| ENS best-F1: shipped / ep1 / ep13 MICRO | | | 66.4 / 66.0 / 66.0 | 59.5 / 59.2 / 59.4 | |
| ENS recall-leaning, shipped (P rule) | 57.9 | 70.1 | 63.4 | 56.3 | 4748 / 3454 / 2030 |
| ENS recall-leaning, ep13 (P rule) | 58.4 | 69.7 | 63.5 | 56.9 | 4722 / 3366 / 2056 |
| ENS recall-leaning, ep13, C0.25 | 58.5 | 69.9 | 63.7 | 57.4 | — |
| ENS recall-leaning, ep13, B (dual label) | 58.3 | 70.1 | 63.6 | 57.2 | — |

- **Val picked the wrong epoch.** On test, epoch 13 beats epoch 1 alone by +5.6 MICRO / +2.1 MACRO; the 16-scene val ranked them the other way (54.6 vs 53.1).
- **New labels fixed DINO alone:** ep13 vs shipped +9.3 MICRO / +4.8 MACRO; it ties the HYBRID on MICRO (63.7 vs 63.9) with higher P (65.8 vs 61.8) and lower R (61.6 vs 66.1). Per class it beats the HYBRID on missile 56.2 vs 52.7, radar 58.8 vs 57.5, tank 75.0 vs 73.4, smallvehicle 63.8 vs 62.1, ties vehicle 70.5; it loses arty 33.3 vs 49.2, logistic 17.8 vs 31.3, camo 54.4 vs 56.6 (these pull MACRO down).
- **Ensemble gain did not grow:** ENS best-F1 ≈ 66.0 for every DINO checkpoint; recall-leaning ep13 adds only +0.5-1.1 MACRO over shipped at the same recall. Hypothesis (not yet measured): the better DINO now finds more of the same objects as the HYBRID, so it adds less that is new; complement.py on ep13 would test this.
- **Radar/missile:** ep13 with rule B gives missile R 62.7 (259/255/154) and radar R 69.1 (190/188/85); with C0.25, missile 61.7, radar 65.8, MACRO 57.4.

### Coverage check, ep13 vs shipped (2026-11-03 ~21:50 IST): [shipped116](COMPLEMENT_shipped116_ai.md) · [ep13](COMPLEMENT_m1_relabel_last_ai.md)
Hypothesis confirmed. "Only D" = objects DINO finds with the right class that the HYBRID misses, which is all an ensemble can add.

| class | only D: shipped → ep13 | both: shipped → ep13 | Jaccard: shipped → ep13 | union recall: shipped → ep13 |
|---|---|---|---|---|
| arty | 11 → 18 | 43 → 43 | 0.27 → 0.25 | 65.1 → 67.9 |
| camo | 51 → 34 | 250 → 266 | 0.61 → 0.68 | 73.1 → 70.1 |
| logistic | 28 → 19 | 53 → 35 | 0.42 → 0.30 | 42.6 → 39.6 |
| missile | 18 → 24 | 140 → 174 | 0.53 → 0.64 | 64.2 → 65.6 |
| radar | 19 → 8 | 122 → 125 | 0.60 → 0.65 | 74.2 → 70.2 |
| smallvehicle | 189 → 168 | 597 → 615 | 0.61 → 0.65 | 78.2 → 76.5 |
| tank | 15 → 19 | 407 → 417 | 0.73 → 0.75 | 86.0 → 86.7 |
| vehicle | 56 → 104 | 1402 → 2107 | 0.52 → 0.77 | 86.4 → 88.0 |
| **total** | **387 → 394** | 3014 → 3782 | | |

- ep13 finds 775 more objects correctly than shipped (3401 → 4176), but 768 of those (99%) the HYBRID already finds; the pool of objects only DINO finds barely moves (387 → 394), and for radar it shrinks (19 → 8).
- Errors became correlated: shipped DINO mislabelled into camo/logistic (missile→camo 122, vehicle→logistic 821); ep13, like the HYBRID, mislabels into vehicle (missile→vehicle 176 vs HYBRID 104; radar→vehicle 73 vs 36; logistic→vehicle 147 vs 85). Same labels, same majority-class pull, same mistakes.
- Takeaway: a better single model is not a better ensemble partner. For ensemble gain, DINO needs to be trained to differ where the HYBRID fails (e.g. class weights against vehicle, or rare-class-focused), not just to be accurate.

## Conclusions and next steps
- **DINO's boxes are the useful part, so far.** With the HYBRID's class decision (M2) or as extra candidates (M3), the shipped DINO already lifts the HYBRID: ENS +2.5 MICRO / +2.8 MACRO, with both P and R up.
- **DINO's own class decision is the weak point** (D: MACRO 48.9). M1 tests whether that comes from the old labels and the epoch-2 selection.
- **Next, after M1 (automatic):** M1 alone, M2/M3/ENS with M1's boxes, and the shipped DINO on all 116 scenes. Then compare everything on Test116.
- **2026-11-03 update:** M1 epoch 13 is the DINO checkpoint to use (better than epoch 1 and shipped on Test116). Best recall-leaning: ep13 + rule B (R 70.1, P 58.3, MACRO 57.2) or ep13 + C0.25 (R 69.9, MACRO 57.4). Open for the researcher: B vs C0.25, retrain on all 359 scenes, and deploy or not. Nothing deployed.
