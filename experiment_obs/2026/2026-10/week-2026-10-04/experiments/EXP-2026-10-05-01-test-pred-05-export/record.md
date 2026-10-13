# EXP-2026-10-05-01-test-pred-05-export

Status: completed (README inside zip contains one error).
Stage2 cross-reference: **e0004** (ID used before the notebook migration). Actual work date: 2026-10-05 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/test_pred_05.zip`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Deliver every ensemble prediction with score ≥ 0.05 on test_80 as JSON.

## Actual setup and changes

Source `predictions_final_c05` (per-class, no dedup); polygons in native TIFF px and 0.30 m px.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-10-05-01-test-pred-05-export` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical: JSON built from c05 files and zipped.

## Results and failures

23,041 boxes (vehicle 9,454 · smallvehicle 4,606 · camo 2,365 · logistic 2,017 · tank 1,943 · arty 987 · missile 981 · radar 688). GT coverage at IoU 0.3: 89.4 % (all 80) / 89.5 % (79 usable); correct-class box 79.0 %.

## Supporting evidence

Attached here: [`zip-README.md`](zip-README.md) (as shipped, contains the delete-region error).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-10-05-01-test-pred-05-export` — 2.8 MB.

## Conclusions and next steps

**Error:** the zip README says delete-region boxes were dropped — they were not; 4,194 boxes in 24 scenes have their centre in a delete region (the infer_test ignore filter read a non-existent folder). Box data are correct.
