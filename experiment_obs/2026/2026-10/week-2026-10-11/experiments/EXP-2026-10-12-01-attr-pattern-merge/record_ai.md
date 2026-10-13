# EXP-2026-10-12-01-attr-pattern-merge — AI-authored

Author: AI agent (Claude). Human/legacy companion: [record.md](record_ai.md).
Moved here on 2026-10-14 from `record.md` under the human/AI authorship rule (content unchanged except links).

Status: running (missile row locked 2026-10-13; radar row awaiting user decisions).
Stage2 cross-reference: **e0007** (ID used before the notebook migration). Actual work date: 2026-10-12 (Asia/Kolkata).

## Request summary and authorization

Imported on 2026-10-14 from earlier sessions (original folder `/sfs/weekly_exp/oct_w3/n0001_attr_pattern_merge`). The work itself was requested and authorized in those sessions; this record summarizes it.

## Question and design

Which prediction errors are 'logically acceptable' given fine labels/attributes, and what operational rule set (P03) should score stage 2?

## Actual setup and changes

5,308 test instances × attributes + Parent Class (XML) joined to m0001 deploy boxes, 8-class vectors and c05 evidence; rule engine `rules.py`; per-class traces; user rule scripts `missile_rule_v2.py`, `radar_rule_v1.py`. Env `/sfs/envs/tide`.

- Code/artifacts: `/sfs/stage2/artifacts/EXP-2026-10-12-01-attr-pattern-merge` (not synchronized). Research Git: 826651e (research Git /sfs/stage2, dirty: import + migration edits uncommitted at record time).
- Data: `/sfs/data/up42_platform_data_27_08_with_soc` (test_80 = Test1 43 + Test2 37); stage-2 derived data `/sfs/stage2/data/derived/`.

## Execution

2026-10-12/13/14 in this session (scripts in artifacts). Waterfall: `round2.py`, `round3_pairs.py`, `round4_stack.py`, `make_charts.py`, `write_report.py`.

## Results and failures

Proposed stack F1 56.6 → 70.6 (all rules) — user rejected E2/E5/E10 and deferred 2nd labels. **Missile row (P03, locked):** first label only; P_Camo → missile|camo; camo-parent missile hardware → missile|camo; Is_Empty Yes/Nil → missile|vehicle; loaded → missile only ⇒ **recall 67.1 % (214/319), precision 72.1 % (160/222), F1 69.5**. 40 missile false positives sit on large trucks (10–18 m) inside missile sites. **Radar row (pending):** same principles ⇒ recall 56.7 % (118/208), precision 50.0 % (112/224); 66 radar boxes on no object; options EWS→vehicle, stowed→vehicle, helicopter-neutral up to R 65.9 / P 51.1.

## Supporting evidence

Attached here: [`REPORT.md`](REPORT.md), [`chart_waterfall_f1.png`](chart_waterfall_f1.png), [`chart_per_class.png`](chart_per_class.png), [`round2.json`](round2.json), [`round4_stack.json`](round4_stack.json), [`pair_thresholds.tsv`](pair_thresholds.tsv), [`missile_fp_on_vehicles.tsv`](missile_fp_on_vehicles.tsv).

External (kept outside the notebook): `/sfs/stage2/artifacts/EXP-2026-10-12-01-attr-pattern-merge` — 5.1 MB.

## Conclusions and next steps

Next: user decisions on radar (EWS, stowed Closed/Nil, helicopters); then remaining classes. Open visual checks: 8 P_Camo-loaded missiles called vehicle, 31 MLRS via Is_Empty=Nil, 40 missile boxes on trucks.
