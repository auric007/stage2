# EXP-2026-10-14-07-ens-c-y8mft-y11l — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T03:37:48+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-07-ens-c-y8mft-y11l`

Status: queued (after the control; inference only).

## Request summary and authorization

Agent-chosen follow-up within the researcher's overnight authorization (2026-10-14 ~02:00 IST: keep the GPU busy with experiments chosen from the analysis until stopped).

## Question and design

Deployable view of variant **C**: ensemble of the variant's fine-tuned YOLOv8m ([EXP-2026-10-14-02-relabel-c-unloaded-vehicle](../EXP-2026-10-14-02-relabel-c-unloaded-vehicle/record_ai.md)) with m0001's **unchanged** YOLO11l,
same TTA + noisy-or + m0001 deploy thresholds as m0001. Compared with the m0001 ensemble row and with the other variants' ensembles (06–09).
Motivation: B's single-model result (missile P03 F1 +3.5, overall −1.5 vs single baseline) — does the gain survive in the deployed ensemble form?

## Actual setup and changes

Weights: `/sfs/stage2/artifacts/EXP-2026-10-14-02-relabel-c-unloaded-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt`. Pipeline `code/run_experiment.sh` with epochs 0 (code commit `1b03951`). V100.
Note: the automatic results table labels the row "single model + TTA"; for this experiment it is the **2-model ensemble**.

## Execution

`bash /sfs/stage2/code/run_experiment.sh EXP-2026-10-14-07-ens-c-y8mft-y11l C /sfs/stage2/artifacts/EXP-2026-10-14-02-relabel-c-unloaded-vehicle/runs/ft/weights/best.pt,/sfs/stage2/models/BEST/prod_y11l_best.pt 0`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.
