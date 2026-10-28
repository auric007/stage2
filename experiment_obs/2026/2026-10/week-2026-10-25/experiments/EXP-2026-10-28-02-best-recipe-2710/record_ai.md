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

## Results

Pending.
