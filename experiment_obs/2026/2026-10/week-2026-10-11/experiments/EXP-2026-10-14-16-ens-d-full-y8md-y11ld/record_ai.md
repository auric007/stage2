# EXP-2026-10-14-16-ens-d-full-y8md-y11ld — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-16-ens-d-full-y8md-y11ld`

Status: queued.

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

**Full-D ensemble**: D YOLOv8m (EXP-03) + D YOLO11l (EXP-13) (inference only). Does a fully relabelled ensemble beat m0001 on the missile row and overall?
Compare with EXP-08 (D y8m + original y11l) and EXP-10 (m0001 same pipeline).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/artifacts/EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt,/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_D` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `none`.

## Execution

Queue line: `EXP-2026-10-14-16-ens-d-full-y8md-y11ld|D|/sfs/stage2/artifacts/EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt,/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt|0||`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-14 21:19 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-16-ens-d-full-y8md-y11ld** (variant D) | 52.1 / 59.8 | 73.6 / 56.7 | 68.0 / 72.1 | 70.4 / 64.7 | 55/103 | 57.2 / 45.9 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 33/37 · camo 45/36 · logistic 29/26 · missile 31/41 · radar 37/51 · smallvehicle 40/58 · tank 68/64 · vehicle 62/76

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-16-ens-d-full-y8md-y11ld` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
