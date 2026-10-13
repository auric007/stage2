# EXP-2026-10-14-04-control-finetune-a-unchanged — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T02:12:37+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-04-control-finetune-a-unchanged`

Status: queued (overnight GPU queue `code/queue/queue.txt`; results section is appended automatically when it finishes).

## Request summary and authorization

2026-10-14 ~02:00 IST: researcher authorized the overnight GPU queue — start with the three missile relabel experiments (B, C, D) as single-YOLOv8m runs with m0001's y8m as baseline, then keep the GPU busy with agent-chosen follow-ups until stopped in the morning. Control for B/C/D (proposed by the agent, within the overnight authorization): identical 10-epoch fine-tune on the **unchanged** labels.

## Question and design

How much of any B/C/D change comes from the extra fine-tuning itself rather than the relabelling? Variant **A** (`engine_v1/build_variant_labels.py`; variant A reproduces m0001's training labels byte-for-byte: 0 / 14,372 files differ).
Compared against: EXP-2026-10-14-05 (same pipeline, no fine-tune) and EXP-2026-10-14-04 (same fine-tune, unchanged labels).
Metrics: P01 deploy P/R and recall-first; missile row under P03 (D-20261013-02) and under the variant's own definition; share of relabelled objects predicted `vehicle`; radar row.
Success criterion: missile P03 F1 above the control's without losing ≥ 2 points of P01 deploy F1.

## Actual setup and changes

Dataset `data/derived/train_final` (m0001's own training set).
- Code: research Git `/sfs/stage2` commit `1b03951` (clean for the pipeline files). Notebook revision: see sync log.
- Pipeline: `code/run_experiment.sh` → fine-tune (`engine_v1/train_finetune.py`: init `/sfs/stage2/models/BEST/prod_y8m_best.pt` = m0001's YOLOv8m member, 10 epochs, AdamW lr0 2e-4 → lrf 0.1, warmup 0.5, close_mosaic 2, imgsz 1024, batch 16, seed 0, m0001 augmentation) → single-model TTA inference (1024/1536 × none/h, noisy-or, NMS 0.4, conf 0.05) → deploy post-processing with m0001 thresholds → P01 reports → rule rows (`engine_v1/score_rules.py`) → automatic results section (`code/queue/publish_result.py`).
- Env `/sfs/envs/yolo_models` (ultralytics 8.4.41, torch 2.4.0+cu124); GPU Tesla V100-PCIE-32GB (probe: ~2 it/s, 15.7 GB at batch 16 → ~8 min/epoch incl. val).
- Test set test_80 (Test1 43 + Test2 37); validation during training = the variant's relabelled test tiles (same convention as m0001).

## Execution

Queued after B, C, D. Command: `bash /sfs/stage2/code/run_experiment.sh EXP-2026-10-14-04-control-finetune-a-unchanged A /sfs/stage2/models/BEST/prod_y8m_best.pt 10`.
Logs: `/sfs/stage2/artifacts/EXP-2026-10-14-04-control-finetune-a-unchanged/logs/` (pipeline, train, infer, score).

## Results and failures

Pending (appended automatically below).

## Conclusions and next steps

Pending.
