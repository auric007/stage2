# EXP-2026-10-14-10-m0001-ens-same-pipeline — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T15:56:23+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-10-m0001-ens-same-pipeline`

Status: queued (inference only, ~20 min).

## Request summary and authorization

Agent follow-up under the researcher's request to keep the GPU busy (2026-10-14 ~02:00) and their 15:55 "update" request.

## Question and design

Like-for-like reference: m0001's unchanged ensemble (y8m + y11l) through the **same** pipeline as EXP-2026-10-14-06…09
(`infer_test.py` TTA → `apply_priority.py` → `report_from_preds.py` → `score_rules.py`). The m0001 numbers quoted so far came from the
original Sept pipeline (`predictions_deploy`); this checks they match before comparing variant ensembles with m0001.

## Actual setup and changes

Weights `/sfs/stage2/models/BEST/prod_y8m_best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt`. Code commit `1b03951`. V100.

## Execution

`bash /sfs/stage2/code/run_experiment.sh EXP-2026-10-14-10-m0001-ens-same-pipeline A /sfs/stage2/models/BEST/prod_y8m_best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt 0`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.

## Results — published 2026-10-14 16:33 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-10-m0001-ens-same-pipeline** (variant A) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | 53.9 / 72.1 | 0/0 | 56.7 / 50.0 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 41/36 · camo 44/35 · logistic 30/25 · missile 30/41 · radar 41/52 · smallvehicle 41/61 · tank 68/65 · vehicle 64/75

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-10-m0001-ens-same-pipeline` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
