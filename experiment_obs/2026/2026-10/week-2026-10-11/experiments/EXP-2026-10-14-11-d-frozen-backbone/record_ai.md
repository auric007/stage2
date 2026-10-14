# EXP-2026-10-14-11-d-frozen-backbone — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-11-d-frozen-backbone`

Status: queued.

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

Variant D with the YOLOv8m **backbone frozen** (first 10 layers). The plain fine-tunes lost ~2 points of overall deploy F1 (control vs baseline). Does freezing the backbone keep the other classes stable while still learning D's missile relabel?
Compare with EXP-2026-10-14-03 (D, unfrozen) and EXP-2026-10-14-04 (control). Success: missile P03 F1 ≥ D's 67.9 with overall deploy F1 closer to the baseline's 57.1.

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/models/BEST/prod_y8m_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_D` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `FREEZE=10`.

## Execution

Queue line: `EXP-2026-10-14-11-d-frozen-backbone|D|/sfs/stage2/models/BEST/prod_y8m_best.pt|10||FREEZE=10`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-14 17:25 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-11-d-frozen-backbone** (variant D) | 56.4 / 56.3 | 70.2 / 51.2 | 52.4 / 86.3 | 49.1 / 80.3 | 63/103 | 50.0 / 46.8 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 47/28 · camo 50/36 · logistic 28/18 · missile 33/24 · radar 38/44 · smallvehicle 43/58 · tank 71/57 · vehicle 66/74

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-11-d-frozen-backbone` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
