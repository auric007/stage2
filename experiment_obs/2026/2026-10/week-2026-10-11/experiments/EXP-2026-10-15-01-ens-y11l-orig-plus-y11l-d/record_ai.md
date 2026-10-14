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
