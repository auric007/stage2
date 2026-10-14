# EXP-2026-10-14-08-ens-d-y8mft-y11l — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T03:37:48+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-08-ens-d-y8mft-y11l`

Status: queued (after the control; inference only).

## Request summary and authorization

Agent-chosen follow-up within the researcher's overnight authorization (2026-10-14 ~02:00 IST: keep the GPU busy with experiments chosen from the analysis until stopped).

## Question and design

Deployable view of variant **D**: ensemble of the variant's fine-tuned YOLOv8m ([EXP-2026-10-14-03-relabel-d-both-vehicle](../EXP-2026-10-14-03-relabel-d-both-vehicle/record_ai.md)) with m0001's **unchanged** YOLO11l,
same TTA + noisy-or + m0001 deploy thresholds as m0001. Compared with the m0001 ensemble row and with the other variants' ensembles (06–09).
Motivation: B's single-model result (missile P03 F1 +3.5, overall −1.5 vs single baseline) — does the gain survive in the deployed ensemble form?

## Actual setup and changes

Weights: `/sfs/stage2/artifacts/EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt`. Pipeline `code/run_experiment.sh` with epochs 0 (code commit `1b03951`). V100.
Note: the automatic results table labels the row "single model + TTA"; for this experiment it is the **2-model ensemble**.

## Execution

`bash /sfs/stage2/code/run_experiment.sh EXP-2026-10-14-08-ens-d-y8mft-y11l D /sfs/stage2/artifacts/EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt 0`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-14 08:07 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-08-ens-d-y8mft-y11l** (variant D) | 53.7 / 59.2 | 75.0 / 58.5 | 68.7 / 69.1 | 70.4 / 60.1 | 51/103 | 57.7 / 48.7 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 37/36 · camo 46/36 · logistic 30/24 · missile 29/42 · radar 39/52 · smallvehicle 42/58 · tank 70/63 · vehicle 64/76

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-08-ens-d-y8mft-y11l` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
