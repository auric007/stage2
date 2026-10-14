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
