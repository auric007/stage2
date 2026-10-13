# EXP-2026-10-08-01-imagewise-deploy — AI-authored

Author: AI agent (Claude). Human/legacy companion: [record.md](record_ai.md).
Moved here on 2026-10-14 from `record.md` under the human/AI authorship rule (content unchanged except links).

Status: completed.
Stage2 cross-reference: **e0005** (ID used before the notebook migration). Actual work date: 2026-10-08 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/oct_w2/n0001_imagewise_deploy`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Per-image confusion matrix, per-class P/R and an instance-ID table for all 80 test images at the deployment thresholds.

## Actual setup and changes

P01 rules: simple mapping, aerial/radome/concealment dropped, IoU 0.3, reference per-class first-match rule; instance IDs from `label_detail_id` col 26.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-10-08-01-imagewise-deploy` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical: `build.py` (in artifacts).

## Results and failures

Totals reproduce deploy STATS exactly (TP 3,059 / FP 2,646 / GT 5,100). FN 2,041 = 1,123 wrong class + 918 no box; FP 2,646 = 1,123 wrong-class + 1,523 false alarms. Median image P 50.7 / R 58.5.

## Supporting evidence

Attached here: [`original-README.md`](original-README.md), [`SUMMARY.md`](SUMMARY.md), [`summary.json`](summary.json), [`images/`](images/).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-10-08-01-imagewise-deploy` — 0.7 MB.

## Conclusions and next steps

Per-image files attached.
