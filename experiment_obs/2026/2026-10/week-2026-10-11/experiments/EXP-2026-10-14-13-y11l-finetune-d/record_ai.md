# EXP-2026-10-14-13-y11l-finetune-d — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d`

Status: queued.

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

m0001's **YOLO11l** member fine-tuned on variant D labels (batch 12 for V100 memory). With only the y8m relabelled, ensemble D (EXP-08) was level with m0001. Does relabelling both members give a full-D ensemble that beats m0001 on missiles?
Single-model numbers here; the ensemble is EXP-2026-10-14-16.

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/models/BEST/prod_y11l_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_D` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `BATCH=12`.

## Execution

Queue line: `EXP-2026-10-14-13-y11l-finetune-d|D|/sfs/stage2/models/BEST/prod_y11l_best.pt|10||BATCH=12`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-14 20:23 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-13-y11l-finetune-d** (variant D) | 58.9 / 54.7 | 72.1 / 52.7 | 59.6 / 89.3 | 58.8 / 82.1 | 57/103 | 51.0 / 53.6 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 34/18 · camo 51/33 · logistic 31/20 · missile 37/32 · radar 44/46 · smallvehicle 48/55 · tank 65/64 · vehicle 70/69

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
