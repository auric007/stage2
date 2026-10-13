# EXP-2026-10-09-01-detailed-camo-credit — AI-authored

Author: AI agent (Claude). Human/legacy companion: [record.md](record_ai.md).
Moved here on 2026-10-14 from `record.md` under the human/AI authorship rule (content unchanged except links).

Status: completed.
Stage2 cross-reference: **e0006** (ID used before the notebook migration). Actual work date: 2026-10-09 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/oct_w2/n0002_detailed_camo_credit`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Score deploy predictions on labels_detailed with camo credit (protocol P02).

## Actual setup and changes

Camouflaged object correct if predicted camo OR hardware class; recall row = hardware class; global best-IoU matching.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-10-09-01-detailed-camo-credit` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

Historical: `score.py` (in artifacts).

## Results and failures

MICRO **P 57.6 / R 64.4** (strict labels_detailed without credit: P 55.5 / R 62.1). Missile P 30.2 → 72.1.

## Supporting evidence

Attached here: [`REPORT.md`](REPORT.md), [`results.json`](results.json).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-10-09-01-detailed-camo-credit` — 0.0 MB.

## Conclusions and next steps

Became protocol P02; superseded for missile by the user rule set (P03).
