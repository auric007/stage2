# EXP-2026-10-14-18-y11l-control-finetune-a — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T21:22:07+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-18-y11l-control-finetune-a`

Status: completed (2026-10-14 21:33–23:18 IST; train 94 min).

## Request summary and authorization

Agent follow-up under the researcher's standing request to keep the GPU busy with analysis-driven experiments (2026-10-14 ~02:00; follow-ups approved ~16:10). Chosen 21:20 after EXP-2026-10-14-13 (YOLO11l on D) gave the best missile row so far.

## Question and design

YOLO11l fine-tuned 10 epochs on **unchanged** labels (control for EXP-13 and EXP-19). Same role as EXP-2026-10-14-04 for the y8m: separates the fine-tune effect from the relabel effect.
Compare with EXP-2026-10-14-13 (y11l on D), EXP-2026-10-14-10 (m0001 same pipeline).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 → `score_rules.py`). Fine-tune recipe as EXP-2026-10-14-13 (10 epochs, AdamW lr0 2e-4, batch 12, imgsz 1024, seed 0). Code commit: see daily log. GPU V100. Init `/sfs/stage2/models/BEST/prod_y11l_best.pt`; data `data/derived/train_variant_A (train_final)`; extra env `BATCH=12`.

## Execution

Queue line: `EXP-2026-10-14-18-y11l-control-finetune-a|A|/sfs/stage2/models/BEST/prod_y11l_best.pt|10||BATCH=12`

## Results and failures

See the automatic results section at the end. No failures.

## Conclusions and next steps

Control fine-tune of the y11l on unchanged labels: deploy F1 57.2 (−0.6 vs original), missile P03 F1 69.5, radar 50.9 / 54.4 (−5.0 / −5.4). The fine-tune itself costs radar quality. vs this control, D (EXP-13) gives missile +2.0 F1 (P +10.5) at deploy −0.5.

## Results — published 2026-10-14 23:18 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-18-y11l-control-finetune-a** (variant A) | 60.4 / 54.4 | 73.0 / 54.5 | 62.1 / 78.8 | 50.8 / 78.8 | 0/0 | 50.0 / 51.9 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 43/18 · camo 52/34 · logistic 32/17 · missile 32/37 · radar 42/44 · smallvehicle 51/56 · tank 65/65 · vehicle 72/68

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-18-y11l-control-finetune-a` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
