# EXP-2026-10-14-15-ens-r-y8mft-y11l — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-15-ens-r-y8mft-y11l`

Status: completed (2026-10-14, see runner log).

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

Ensemble: variant-R YOLOv8m (EXP-12) + m0001's unchanged YOLO11l (inference only). Deployable form of EXP-12.
Compare with EXP-2026-10-14-09 (control ensemble).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/artifacts/EXP-2026-10-14-12-radar-r-stowed-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_R` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `none`.

## Execution

Queue line: `EXP-2026-10-14-15-ens-r-y8mft-y11l|R|/sfs/stage2/artifacts/EXP-2026-10-14-12-radar-r-stowed-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt|0||`

## Results and failures

See the automatic results section at the end. No failures.

## Conclusions and next steps

Ensemble R y8m + original y11l: deploy F1 55.7; missile P03 F1 65.9; radar F1 51.5 off / 54.5 on (P 49.3). The radar-precision gain of single R (63.8) does not survive the ensemble because the y11l was not trained on R.

## Results — published 2026-10-14 21:00 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-15-ens-r-y8mft-y11l** (variant R) | 53.0 / 58.7 | 73.3 / 52.7 | 69.3 / 62.8 | 58.6 / 62.8 | 0/0 | 53.8 / 49.3 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 32/37 · camo 46/36 · logistic 29/23 · missile 27/45 · radar 40/48 · smallvehicle 42/58 · tank 69/62 · vehicle 64/75

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-15-ens-r-y8mft-y11l` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
