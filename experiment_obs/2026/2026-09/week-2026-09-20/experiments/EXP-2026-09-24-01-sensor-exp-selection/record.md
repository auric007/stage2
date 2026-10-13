# EXP-2026-09-24-01-sensor-exp-selection

Status: completed (selection); arm B training not started.
Stage2 cross-reference: **e0002** (ID used before the notebook migration). Actual work date: 2026-09-24 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/sep_w1/sensor_exp`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Does the training sensor matter? Pick 40 train images for same-area re-collection from a different sensor (A = current, B = re-collected).

## Actual setup and changes

Object embeddings from m0001 prod_y8m neck (P3+P4 ROI-mean, centred); class-conditional facility-location selection, ≥12 sparse rare-class images, ≤10 dense, site cap 2 km; 25-sensor list.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-09-24-01-sensor-exp-selection` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical: `embed_objects.py`, `select_images.py` (in artifacts).

## Results and failures

SELECTED_40 (+RESERVE_10); covers 56 % of the gap between a random 40 and all 316 train images on the test-coverage objective. 723 small vehicles in the 40 (all parent class 00).

## Supporting evidence

Attached here: [`original-README.md`](original-README.md), [`SELECTED_40.txt`](SELECTED_40.txt), [`SELECTED_40.tsv`](SELECTED_40.tsv), [`RESERVE_10.txt`](RESERVE_10.txt), [`RESERVE_10.tsv`](RESERVE_10.tsv), [`SENSOR_LIST.txt`](SENSOR_LIST.txt).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-09-24-01-sensor-exp-selection` — 27.9 MB.

## Conclusions and next steps

Waiting for analyst re-collection; arm B needs re-annotation.
