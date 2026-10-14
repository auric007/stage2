# EXP-2026-10-15-02-y11l-finetune-d-lowlr — AI-authored

Author: AI agent (Claude). Generated: 2026-10-15T01:05:21+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-15-02-y11l-finetune-d-lowlr`

Status: queued.

## Request summary and authorization

Agent follow-up under the researcher's standing request to keep the GPU busy with analysis-driven experiments (2026-10-14). Chosen 2026-10-15 01:05 after EXP-2026-10-14-19.

## Question and design

YOLO11l fine-tuned on D with a **4× lower learning rate** (lr0 5e-5 instead of 2e-4), else as EXP-13. Every fine-tune so far lowered overall deploy F1 vs its starting model (y11l 57.8 → control 57.2 → D 56.7). Does a gentler fine-tune keep D's missile gain with less drift? Success: missile F1 ≥ 71 with deploy F1 ≥ 57.5.
Compare with EXP-2026-10-14-17 (original y11l), -13 (y11l on D), -18 (y11l control), -10 (m0001).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA, noisy-or, m0001 deploy thresholds, `score_rules.py`). Weights/init `/sfs/stage2/models/BEST/prod_y11l_best.pt`; extra env `BATCH=12 LR0=0.00005`. GPU V100.

## Execution

Queue line: `EXP-2026-10-15-02-y11l-finetune-d-lowlr|D|/sfs/stage2/models/BEST/prod_y11l_best.pt|10||BATCH=12 LR0=0.00005`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.
