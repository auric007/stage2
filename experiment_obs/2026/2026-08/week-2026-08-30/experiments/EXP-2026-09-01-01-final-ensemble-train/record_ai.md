# EXP-2026-09-01-01-final-ensemble-train — AI-authored

Author: AI agent (Claude). Human/legacy companion: [record.md](record_ai.md).
Moved here on 2026-10-14 from `record.md` under the human/AI authorship rule (content unchanged except links).

Status: completed.
Stage2 cross-reference: **e0001** (ID used before the notebook migration). Actual work date: 2026-09-01 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/sep_w1 (sensor_exp split off)`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Build one deployable 8-class oriented-box detector (no SOC segregation) with the best July recipe: ensemble + TTA + Google-Earth booster on the full 27_08 data.

## Actual setup and changes

YOLOv8m-OBB (`prod_y8m`, 68 epochs) + YOLO11l-OBB (`prod_y11l`, 78 epochs), 100 ep / patience 30, scale-aug 0.3, AdamW; data `data/derived/train_final` (27_08 Train 10,758 tiles + GE booster 918 canvases, 0.30 m GSD, 1024 / 20 %); val = test tiles 2,696. Env `/sfs/envs/yolo_models` (ultralytics 8.4.41, torch 2.4.0). GPU L4 then L40S (resumed from checkpoint at epoch 31). Inference: TTA scales 1024/1536 × flips none/h, per-tile noisy-or → scene max, NMS 0.4.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical (Sep 1–2, 2026): `run_final.sh`, `run_final_resume.sh` (see attached original README). Later deliverables in the same lab: deploy post-processing (`engine/apply_priority.py` → `predictions_deploy`), 8-class score vectors and top-k analysis, standalone inference package `infer_point_stage2/`, 5-scene bundle, rank-1/rank-2 JSON.

## Results and failures

test_80, IoU 0.3: recall-first (P ≥ 0.30) **MICRO 74.4 / MACRO 58.0**; IoU 0.5 72.4 / 55.6. Deploy thresholds (radar 0.39, missile 0.85, arty 0.86, logistic 0.77, camo 0.42, smallvehicle 0.24, tank 0.54, vehicle 0.84; dedup 0.4, priority radar/missile/arty/logistic): **P 53.6 / R 60.0** (TP 3,059 · FP 2,646 · GT 5,100). Top-k: true class in top-3 for 87.1 % vs top-1 69.8 %; 93.4 % of objects get a box. Failure noted: `predictions_final/` is empty (infer_test read GT wrongly → thresholds 1.0) — use `predictions_final_c05` / `predictions_deploy`.

## Supporting evidence

Attached here: [`original-README.md`](original-README.md), [`FINAL_ENSEMBLE.md`](FINAL_ENSEMBLE.md), [`FINAL_REPORT_DETAILED.md`](FINAL_REPORT_DETAILED.md), [`FINAL_SCOREWISE.md`](FINAL_SCOREWISE.md), [`FINAL_TOPK.md`](FINAL_TOPK.md), [`deploy-STATS.md`](deploy-STATS.md), [`MODEL_CARD.md`](MODEL_CARD.md).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train` — 901.0 MB.
  - `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/FINAL/prod_y8m_best.pt` — 53.6 MB, sha256 c186bb958d4211d6…
  - `/sfs/stage2/artifacts/EXP-2026-09-01-01-final-ensemble-train/FINAL/prod_y11l_best.pt` — 53.4 MB, sha256 79cdd4cbf3b95527…

## Conclusions and next steps

Promoted as model **m0001** (`/sfs/stage2/models/BEST`). Main residual error is classification, not detection (dominant wrong class: vehicle).
