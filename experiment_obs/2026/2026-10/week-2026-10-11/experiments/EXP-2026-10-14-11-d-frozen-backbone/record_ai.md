# EXP-2026-10-14-11-d-frozen-backbone — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-11-d-frozen-backbone`

Status: queued.

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

Variant D with the YOLOv8m **backbone frozen** (first 10 layers). The plain fine-tunes lost ~2 points of overall deploy F1 (control vs baseline). Does freezing the backbone keep the other classes stable while still learning D's missile relabel?
Compare with EXP-2026-10-14-03 (D, unfrozen) and EXP-2026-10-14-04 (control). Success: missile P03 F1 ≥ D's 67.9 with overall deploy F1 closer to the baseline's 57.1.

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/models/BEST/prod_y8m_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_D` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `FREEZE=10`.

## Execution

Queue line: `EXP-2026-10-14-11-d-frozen-backbone|D|/sfs/stage2/models/BEST/prod_y8m_best.pt|10||FREEZE=10`

## Results and failures

Pending (appended automatically).

## Conclusions and next steps

Pending.
