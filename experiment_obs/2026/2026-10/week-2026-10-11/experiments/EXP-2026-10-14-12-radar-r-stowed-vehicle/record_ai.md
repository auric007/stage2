# EXP-2026-10-14-12-radar-r-stowed-vehicle — AI-authored

Author: AI agent (Claude). Generated: 2026-10-14T16:14:44+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-14-12-radar-r-stowed-vehicle`

Status: completed (2026-10-14 17:25–18:38 IST).

## Request summary and authorization

2026-10-14 ~16:10 IST: researcher approved (a) YOLO11l fine-tuned on D labels for a full-D ensemble, (b) D with a frozen backbone, (c) the stowed-radar rule both as evaluation-only (on/off) and as a training experiment (stowed radars relabelled `vehicle` vs left as `radar`).

## Question and design

Variant **R**: radars with Position Closed or Nil (stowed / not recorded) labelled `vehicle` in training and validation (151 train / 160 val tile instances); missiles unchanged. Researcher's stowed-radar question as a training experiment: does teaching stowed radars as `vehicle` sharpen the radar class?
Control = EXP-2026-10-14-04 (same fine-tune, unchanged labels). Report radar rule off / on / variant-R definition, and the share of stowed radars predicted `vehicle`.

## Actual setup and changes

Pipeline `code/run_experiment.sh` (TTA 1024/1536 × none/h, noisy-or, NMS 0.4, m0001 deploy thresholds → P01 reports → `score_rules.py` rows: missile P03, missile/radar variant definitions, radar rule off (`radar_base`) and stowed rule on (`radar_stowed`)). Fine-tune recipe as EXP-2026-10-14-01 (10 epochs, AdamW lr0 2e-4 → lrf 0.1, imgsz 1024, seed 0, m0001 augmentation). Env `/sfs/envs/yolo_models`; GPU Tesla V100-PCIE-32GB.
Weights/init: `/sfs/stage2/models/BEST/prod_y8m_best.pt`. Variant data: `/sfs/stage2/data/derived/train_variant_R` (built by `engine_v1/build_variant_labels.py`; variant A self-check 0/14,372 files differ). Extra env: `none`.

## Execution

Queue line: `EXP-2026-10-14-12-radar-r-stowed-vehicle|R|/sfs/stage2/models/BEST/prod_y8m_best.pt|10||`

## Results and failures

See the automatic results section at the end. No failures.

## Conclusions and next steps

vs the **control** (EXP-2026-10-14-04, same fine-tune, unchanged labels):
radar rule OFF R 48.1 → 46.2, P 59.1 → **63.8** (F1 53.0 → 53.6); rule ON R 55.3 → 54.3, P 59.1 → 63.8 (F1 57.1 → **58.7**);
overall deploy F1 55.0 → 55.7; missile unaffected (P03 R 66.1 / P 66.5). Stowed radars predicted `vehicle`: 17 / 74 (control 15) —
the relabel barely changed what stowed radars are called, but removing them from the radar class made radar boxes **more precise** (+4.7 P).
Ensemble form queued (EXP-2026-10-14-15).

## Results — published 2026-10-14 18:38 IST (automatic, `code/queue/publish_result.py`)

test_80, IoU 0.3, single model + TTA, deploy thresholds of m0001. P/R in %. Recall-first = mean R@P≥0.30 (micro / macro).

| model | deploy P01 P / R | recall-first micro / macro | missile P03 R / P | missile (variant def) R / P | relabelled → vehicle | radar R / P |
|---|--:|--:|--:|--:|--:|--:|
| **EXP-2026-10-14-12-radar-r-stowed-vehicle** (variant R) | 59.0 / 52.7 | 69.2 / 47.7 | 66.1 / 66.5 | 55.2 / 66.5 | 0/0 | 46.2 / 63.8 |
| single y8m baseline (no fine-tune, same pipeline) | 60.5 / 54.1 | 71.6 / 53.9 | 54.2 / 78.2 | 42.6 / 78.2 | 0/0 | 46.2 / 56.0 |
| m0001 ensemble (reference) | 53.6 / 60.0 | 74.4 / 58.0 | 67.1 / 72.1 | – | – | 56.7 / 50.0 |

Per class, deploy P01 (P / R %): arty 38/30 · camo 51/37 · logistic 34/8 · missile 29/42 · radar 54/40 · smallvehicle 46/52 · tank 70/60 · vehicle 71/66

Evidence: [metrics_ai.json](metrics_ai.json), [P01_detailed_ai.md](P01_detailed_ai.md), [deploy_STATS_ai.md](deploy_STATS_ai.md); artifacts `/sfs/stage2/artifacts/EXP-2026-10-14-12-radar-r-stowed-vehicle` (weights `runs/ft/weights/best.pt`, predictions `preds/`, logs `logs/`).
