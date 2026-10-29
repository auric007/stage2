# EXP-2026-10-28-02-best-recipe-2710 — AI-authored

Author: AI agent (Claude). Generated: 2026-10-28T19:20+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710`

Status: running.

## Request and authorization

- Researcher (2026-10-28 ~19:05 IST): "can you run our best training recipe on our new train/test set and compare it with each other". They shared the current best table (test_80, P04 v2, IoU 0.3) and expect the new split to beat it.
- Chosen option: "Pause the loop after step 10 (~20:15) and run the recipe."
  - The train-subset loop (EXP-2026-10-28-01) was paused with its STOP file at 19:11.
  - Step 10 (87 images, 2 seeds) finishes on its own. The brain judges it when the loop resumes after this run.

## Setup (actual)

- **Recipe:** m0001's (EXP-2026-09-01-01), hyper-parameters unchanged.
  - YOLOv8m-OBB (batch 16) then YOLO11l-OBB (batch 12), from the DOTA-pretrained weights in `/sfs/stage2/models/pretrained`.
  - 100 epochs with patience 30, imgsz 1024, AdamW lr0 = 0.001 × batch / 16, lrf 0.01, warmup 3.
  - Augmentation: degrees 180, scale 0.3, flips 0.5, mosaic 1.0, mixup 0.1, copy_paste 0.1.
  - Script: `code/train_one.py`.
- **Train:** 27_10 Train359 tiled at 0.30 m GSD, 1024 px, step 819.
  - 12,522 tiles, shared with EXP-2026-10-28-01; `labels_simple` under the 2026-10-27 rule.
  - Plus the GE gepack booster: the same 918 canvases m0001 used.
  - 13,440 tiles in total (m0001: 11,676).
- **Validation and best.pt selection:** Test116 tiles (4,003), as m0001 used its test tiles.
  - This selection is on the test set, so best.pt is slightly optimistic. The same applies to m0001 on test_80.
  - last.pt is reported too.
- **Inference:** m0001 pipeline (`engine_v1/infer_test.py`), TTA 1024/1536 × flips none/h, noisy-or, NMS 0.4, conf ≥ 0.05.
- **Scoring** (`code/score_set.py`, identical to EXP-2026-10-28-01):
  - official 27_10 `labels_simple`, IoU 0.3, delete regions dropped;
  - per-class best F1 over conf 0.05–0.90, plus R@P≥0.30, MICRO/MACRO.
- **Scored on:**
  - (a) Test116;
  - (b) the 77 test_80 scenes inside Test116 (of test_80, 2 scenes are now in Train and 1 is not in 27_10).
- **Old models, same pipeline on the same scenes:**
  - m0001 ensemble, m0001 y11l alone, m0001 y8m alone;
  - EXP-2026-10-14-13 (y11l fine-tuned on D);
  - EXP-2026-10-14-14 (frozen-D y8m + y11l ensemble).
  - Together these are the prediction sets behind the researcher's current-best table.
- **Not comparable directly:** the researcher's table is test_80 under P04 v2. Here both old and new models are scored on the same scenes with the same labels.

## Expected time (L40S)

- Inferred from m0001's logs: 215 s/epoch for y8m and 327 s/epoch for y11l, plus about 15% for the larger Train.
  - y8m: ~5–7 h.
  - y11l: ~8–11 h.
  - Inference, scoring and the per-class combination: ~2 h.
- Old-model inference runs in parallel with the y8m training.
- Total ~16–21 h from ~20:15 IST, so results ~12:00–17:00 IST on 2026-10-29.

## Execution log

- 19:16: driver `code/run_all.sh` started. It waits for subset step 10 to finish, then runs the old-model inference in the background and trains y8m → y11l. After that: TTA inference and scoring of new_ens, new_y8m_alone, new_y11l_alone and new_ens_last.
- Smoke test before launch: m0001 ensemble TTA on 2 Test116 scenes, scored with `score_set.py`. Predictions matched the GT (e.g. radar 5/5, missile 3/3), so the coordinate frames agree.

- 20:11: GPU free; y8m training and the old-model inference (m0001 ensemble first) started together.
  - Dataset scan: 13,440 train images (8,364 background tiles, 0 corrupt; 1 duplicate label removed), 4,003 val tiles.
  - Epoch 1: 431 s, about 2× m0001's 215 s, because the GPU is shared with the old-model TTA inference. It should speed up once that finishes. GPU memory 20.8 GB of 46 GB, no OOM.

- 21:04: first old set scored. **old_m0001_ens** (m0001's y8m+y11l, TTA):
  - Test116: MICRO 65.0 / MACRO 56.7. Per-class F1: arty 47.5, camo 53.0, logistic 27.8, missile 59.2, radar 59.0, smallvehicle 60.4, tank 72.5, vehicle 74.0.
  - 77 test_80 scenes inside Test116: MICRO 63.8 / MACRO 56.5.
  - Leak check: no Test116 scene was in the 27_08 Train that m0001 trained on (from `lists/ALL_IMAGES.tsv`: 43 + 34 were in 27_08 Test, 1 in 27_08 no_soc, 6 in 15_10 Test, 32 are new).
  - Epochs 2–3 still ~450 s each while sharing the GPU with inference.

- 21:31: all 5 old sets scored (TTA, labels_simple, IoU 0.3). Test116 MICRO / MACRO, then MICRO / MACRO on the 77 test_80 scenes:
  - old_m0001_ens 65.0 / 56.7 · 63.8 / 56.5
  - old_y11l_alone 64.4 / 54.6 · 62.7 / 54.6
  - old_y8m_alone 62.3 / 54.1 · 61.3 / 53.7
  - old_exp13_y11l_ftD 63.8 / 53.1 · 62.5 / 53.4
  - old_exp14_ens_y8mDfrozen_y11l 65.3 / 56.7 · 64.2 / 56.4
  - Old per-class best on Test116 (max F1 per class over the 5 sets; picked on test, so optimistic, same as the new side will be):
    - arty 47.5 (m0001), camo 53.1 (exp14), logistic 28.6 (exp14), missile 59.2 (m0001), radar 60.5 (y11l), smallvehicle 61.8 (exp13), tank 73.3 (exp14), vehicle 74.0 (m0001).
    - Macro 57.3.

- 01:36 (10-29): new_y8m stopped early at epoch 69 (patience 30), 5.4 h. Best val mAP50 0.443 at epoch 39 (val = Test116 tiles, as in m0001). Clean epochs took ~252 s.
  - new_y11l started 01:36 at ~375 s/epoch.
- 01:52: started `infer_set.sh new_y8m_alone` (best.pt) early, alongside y11l training, for an early read. The driver skips it later because INFER_DONE exists.

- 02:02: **new_y8m_alone** (best.pt, TTA) vs old_y8m_alone. Test116 MICRO / MACRO 63.5 / 55.8 vs 62.3 / 54.1 (+1.2 / +1.7). 77 test_80 scenes 63.4 / 56.1 vs 61.3 / 53.7 (+2.1 / +2.4).
  - Per class on Test116, new vs old:
    - Up: arty 49.9 vs 46.7, camo 53.7 vs 50.3, radar 57.6 vs 53.1, smallvehicle 58.6 vs 56.8, tank 74.2 vs 71.2, vehicle 72.0 vs 71.7.
    - Down or flat: missile 55.3 vs 57.1, logistic 25.5 vs 25.6.
  - Caveat: one seed each. The new best.pt was selected on Test116 tiles, the old one on test_80 tiles, so the new model gets slightly more test-selection optimism on Test116.

- 08:33 (10-29): new_y11l early-stopped at epoch 67 after 6.95 h.
  - Best fitness (0.1·mAP50 + 0.9·mAP50-95) at epoch 37.
  - Note: the y8m 'best epoch 39' quoted above was by mAP50 alone; ultralytics selects best.pt by fitness.
- 09:07: new_ens, new_y11l_alone and new_ens_last inferred and scored; driver finished.
- 11:40: `code/compare.py` → `reports/COMPARISON.md` (full per-class tables with conf/P/R/TP/FP/FN). The subset loop was resumed.

## Results

Full tables: `artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/COMPARISON.md`.

**Test116 (116 scenes), MICRO / MACRO F1:**

| set | Test116 | 77 test_80 scenes |
|---|--:|--:|
| old m0001 ens | 65.0 / 56.7 | 63.8 / 56.5 |
| old exp14 ens | 65.3 / 56.7 | 64.2 / 56.4 |
| **new ens (best.pt)** | 65.0 / **57.4** | **64.7 / 57.5** |
| new ens (last.pt) | 65.1 / 57.0 | 63.7 / 56.5 |
| new y8m alone | 63.5 / 55.8 (old 62.3 / 54.1) | 63.4 / 56.1 (old 61.3 / 53.7) |
| new y11l alone | 63.7 / 56.1 (old 64.4 / 54.6) | 62.6 / 55.9 (old 62.7 / 54.6) |

**Per-class best combination on Test116** (set + conf picked on Test116 itself, so optimistic for both sides):

- Old sets only: MICRO 65.7 / MACRO 57.2.
- New sets only: MICRO 65.9 / MACRO 57.8.
- All sets: MICRO 66.5 / MACRO 58.3.
  - New sets win arty 50.8, camo 55.9, logistic 30.3, tank 74.2 and vehicle 74.0.
  - Old sets win missile 59.2 (m0001), radar 60.5 (old y11l) and smallvehicle 61.8 (exp13).

**Reading:**

- Same recipe, new Train359: MACRO goes up about 0.7–1.7 on every pairing (ensemble +0.7, y8m +1.7, y11l +1.5). MICRO is flat for the ensemble.
- Gains are in arty, camo and tank; logistic gains for y11l.
- Missile and smallvehicle regress slightly; radar is mixed.
- Single seed per model, with no noise estimate. With 0.5–0.7 pt seed noise seen in the proxy, the ensemble's MICRO tie and +0.7 MACRO are within noise; the single-model MACRO gains (+1.5–1.7) are borderline.
- Selection bias: new best.pt is selected on Test116 tiles, old best.pt on test_80 tiles.

## Deployment note (2026-10-29, researcher request)

- The per-class best combination over all 9 sets is the **deployment pick list**:
  - MICRO 66.5 / MACRO 58.3.
  - After cross-class dedup (the product output): **65.8 / 57.7**.
- Full table, weights and inference command: [knowledge/best_combination_ai.md §2](../../../../../knowledge/best_combination_ai.md).
- Machine-readable spec: `artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/combo_test116.json`.
- Per-image robustness (`code/per_image.py` → `reports/PER_IMAGE.md`), conf held fixed, combined / combined + dedup:

| images | MICRO | MACRO | MICRO + dedup | MACRO + dedup |
|---|--:|--:|--:|--:|
| all 116 | 66.5 | 58.3 | 65.8 | 57.7 |
| drop 5 lowest-F1 | 66.8 | 58.7 | 66.0 | 58.0 |
| drop 10 lowest-F1 | 67.8 | 59.3 | 67.0 | 58.7 |
| drop 5 most-errors | 66.6 | 59.7 | 66.4 | 60.2 |
| drop 10 most-errors | 66.6 | 58.8 | 66.2 | 58.9 |

- Median image F1 is 66.7 (p10 36.4, p90 80.0). Dropping the worst 5–10 images adds at most about 1.3 MICRO / 1.4 MACRO.
- Lowest-F1 images are small scenes (1–10 GT) with mostly FPs, plus 23MAY19062258 (121 smallvehicle FN of 190 GT).
- Most-error images are dense scenes (150–460 GT) that also hold many TPs. Dropping them lowers arty to 35.4, because they contain most arty TPs.

## Best-by-conf request (2026-10-29 16:40)

- Researcher asked for every class × conf 0.1..0.9 the set with the best P, the best R and the best F1 (IoU 0.3, all 116 images), over all experiments so far.
- The 16 EXP-2026-10-14/15 sets that had only test_80 predictions are re-inferred on Test116 with the same TTA pipeline: 9 single models + 7 ensembles, 23 passes.
  - Driver: `code/run_old14.sh`. Table: `code/best_by_conf.py`.
- Loop paused by researcher choice, but it had already resolved at 71 images.
