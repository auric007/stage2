# EXP-2026-10-14-13-y11l-finetune-d — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d`

Status: queued.

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

m0001's **YOLO11l** member fine-tuned on variant D labels (batch 12 for V100 memory). With only the y8m relabelled, ensemble D (EXP-08) was level with m0001. Does relabelling both members give a full-D ensemble that beats m0001 on missiles?
Single-model numbers here; the ensemble is EXP-2026-10-14-16.

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/models/BEST/prod_y11l_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_D` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `BATCH=12`.

## Execution

Queue line: `EXP-2026-10-14-13-y11l-finetune-d|D|/sfs/stage2/models/BEST/prod_y11l_best.pt|10||BATCH=12`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.
