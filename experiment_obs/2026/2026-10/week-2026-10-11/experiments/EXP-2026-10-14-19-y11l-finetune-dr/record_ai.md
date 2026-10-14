# EXP-2026-10-14-19-y11l-finetune-dr — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T21:22:07+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-19-y11l-finetune-dr`

Status: queued.

## Request summary and authorization

Agent follow-up under the researcher's standing request to keep the GPU busy with analysis-driven experiments (2026-10-14 ~02:00; follow-ups approved ~16:10). Chosen 21:20 after EXP-2026-10-14-13 (YOLO11l on D) gave the best missile row so far.

## Question and design

YOLO11l fine-tuned on **variant DR** = D missile relabels + R stowed radars → vehicle (518 train / 318 val instances relabelled). Candidate next model: combines the best missile relabel (D) with the radar relabel that raised radar precision (R). Success: missile P03 F1 ≥ 71.5 (EXP-13) and radar F1 above EXP-13's, deploy F1 ≥ m0001 56.6.
Compare with EXP-2026-10-14-13 (y11l on D), EXP-2026-10-14-10 (m0001 same pipeline).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 → `score_rules.py`). Fine-tune recipe as EXP-2026-10-14-13 (10 epochs, AdamW lr0 2e-4, batch 12, imgsz 1024, seed 0). Code commit: see daily log. GPU V100. Init `/sfs/stage2/models/BEST/prod_y11l_best.pt`; data `data/derived/train_variant_DR`; extra env `BATCH=12`.

## Execution

Queue line: `EXP-2026-10-14-19-y11l-finetune-dr|DR|/sfs/stage2/models/BEST/prod_y11l_best.pt|10||BATCH=12`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-15 01:04 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-19-y11l-finetune-dr** (variant DR) | 61.5 / 50.9 | 69.3 / 52.2 | 51.7 / 94.2 | 50.0 / 86.5 | 54/103 | 50.0 / 60.0 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 37/14 · camo 43/40 · logistic 52/17 · missile 38/24 · radar 50/44 · smallvehicle 59/49 · tank 67/65 · vehicle 72/61

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-19-y11l-finetune-dr` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
