# EXP-2026-09-24-02-topk-disagree-export — AI-authored

Author: AI agent (Claude). Human/legacy companion: [record.md](record_ai.md).
Moved here on 2026-10-14 from `record.md` under the human/AI authorship rule (content unchanged except links).

Status: completed.
Stage2 cross-reference: **e0003** (ID used before the notebook migration). Actual work date: 2026-09-24 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/sep_w4/n0001_topk_disagree`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Export test objects where the model's top-1 class ≠ GT but the GT class is ranked 2nd/3rd (possible mislabels or close confusions).

## Actual setup and changes

m0001 per-box 8-class vectors, IoU 0.3, FINAL_TOPK matching rule; native TIFF px, n0005 file format.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-09-24-02-topk-disagree-export` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical: `build.py` (in artifacts).

## Results and failures

879 boxes: tank 209, camo 137, vehicle 137, logistic 104, missile 102, smallvehicle 95, radar 61, arty 34; top confusions tank→vehicle 150, logistic→vehicle 93, smallvehicle→vehicle 86, missile→vehicle 82, camo→tank 77. Counts reproduce FINAL_TOPK exactly.

## Supporting evidence

Attached here: [`original-README.md`](original-README.md), [`summary.csv`](summary.csv), [`stats.json`](stats.json), [`disagreements.csv`](disagreements.csv).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-09-24-02-topk-disagree-export` — 1.5 MB.

## Conclusions and next steps

Label-review candidates; GT-outside-top-3 objects not exported.
