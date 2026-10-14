# EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d — AI-authored

Author: AI agent (Claude). Generated: 2026-10-15T01:05:21+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d`

Status: queued.

## Request summary and authorization

Agent follow-up under the researcher's standing request to keep the GPU busy with analysis-driven experiments (2026-10-14). Chosen 2026-10-15 01:05 after EXP-2026-10-14-19.

## Question and design

Ensemble of the **original YOLO11l** (EXP-17: best deploy F1 57.8 and radar 59.8) and **YOLO11l on D** (EXP-13: best missile F1 71.5), inference only. Can the ensemble keep the original's overall/radar strength and add D's missile precision?
Compare with EXP-2026-10-14-17 (original y11l), -13 (y11l on D), -18 (y11l control), -10 (m0001).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA, noisy-or, m0001 deploy thresholds, `score_rules.py`). Weights/init `/sfs/stage2/models/BEST/prod_y11l_best.pt,/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt`; extra env `none`. GPU V100.

## Execution

Queue line: `EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d|D|/sfs/stage2/models/BEST/prod_y11l_best.pt,/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt|0||`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-15 01:27 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d** (variant D) | 52.7 / 59.4 | 73.4 / 53.2 | 65.5 / 74.6 | 65.3 / 65.7 | 53/103 | 57.7 / 48.3 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 25/24 · camo 47/34 · logistic 27/31 · missile 31/39 · radar 40/52 · smallvehicle 43/58 · tank 64/65 · vehicle 63/76

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-15-01-ens-y11l-orig-plus-y11l-d` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
