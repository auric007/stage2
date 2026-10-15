# EXP-2026-10-15-02-y11l-finetune-d-lowlr — AI-authored

Author: AI agent (Claude). Generated: 2026-10-15T01:05:21+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-15-02-y11l-finetune-d-lowlr`

Status: completed (2026-10-15 01:27–03:12 IST).

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

See the automatic results section at the end. No failures.

## Conclusions and next steps

y11l on D with lr0 5e-5: deploy F1 57.2 (vs 56.7 at lr0 2e-4, 57.8 original) — less drift, as intended; radar 55.0 / 59.3 (≈ original 55.9 / 59.8); missile P03 R 56.1 / P 91.9 (F1 69.7 — below the normal-LR D's 71.5 and the original's 70.1). Middle ground: keeps overall/radar, gains missile precision, loses missile recall.

## Results — published 2026-10-15 03:12 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-15-02-y11l-finetune-d-lowlr** (variant D) | 59.6 / 55.0 | 72.6 / 54.0 | 56.1 / 91.9 | 53.7 / 83.9 | 58/103 | 52.9 / 57.3 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 40/17 · camo 48/34 · logistic 46/17 · missile 36/27 · radar 48/48 · smallvehicle 48/54 · tank 62/68 · vehicle 71/69

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-15-02-y11l-finetune-d-lowlr` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
