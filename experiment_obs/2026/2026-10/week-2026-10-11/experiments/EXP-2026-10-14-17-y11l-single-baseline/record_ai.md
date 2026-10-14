# EXP-2026-10-14-17-y11l-single-baseline — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T21:22:07+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-17-y11l-single-baseline`

Status: queued.

## Request summary and authorization

Agent follow-up under the researcher's standing request to keep the GPU busy with analysis-driven experiments (2026-10-14 ~02:00; follow-ups approved ~16:10). Chosen 21:20 after EXP-2026-10-14-13 (YOLO11l on D) gave the best missile row so far.

## Question and design

m0001's original **YOLO11l alone** (no fine-tune), evaluation only. Isolates how much of EXP-13's missile gain is the YOLO11l itself vs the D relabel + fine-tune.
Compare with EXP-2026-10-14-13 (y11l on D), EXP-2026-10-14-10 (m0001 same pipeline).

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 → `score_rules.py`). Fine-tune recipe as EXP-2026-10-14-13 (10 epochs, AdamW lr0 2e-4, batch 12, imgsz 1024, seed 0). Code commit: see daily log. GPU V100. Init `/sfs/stage2/models/BEST/prod_y11l_best.pt`; data `data/derived/train_variant_A (train_final)`; extra env `none`.

## Execution

Queue line: `EXP-2026-10-14-17-y11l-single-baseline|A|/sfs/stage2/models/BEST/prod_y11l_best.pt|0||`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.
