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

## TODO 3 — instance lists (2026-10-14, Claude; m0001 deploy, P03 missile rule, first label, IoU 0.3)

Researcher TODOs from [daily 2026-10-13](../../daily/2026-10-13.md). Script: `/sfs/stage2/artifacts/EXP-2026-10-12-01-attr-pattern-merge/todo3_instance_lists.py`.

**(a) Missile row, every instance:** [missile_outcomes_ai.tsv](missile_outcomes_ai.tsv) — 319 rows (ID, fine label, Functionality, Is_Empty, P_Camo, Position, Quality, SOC, outcome, predicted class, score, IoU).
Outcomes: correct 214 · missed (no box) 46 · wrong → vehicle 33 · → radar 11 · → arty 6 · → logistic 6 · → camo 3.

Not-correct missiles by fine label:

| fine          |   missed (no box) |   wrong→arty |   wrong→camo |   wrong→logistic |   wrong→radar |   wrong→vehicle |   total |
|:--------------|------------------:|-------------:|-------------:|-----------------:|--------------:|----------------:|--------:|
| HQ9           |                 8 |            0 |            1 |                5 |             1 |              12 |      27 |
| MissileSystem |                 9 |            0 |            1 |                0 |             5 |               2 |      17 |
| HQ12          |                10 |            0 |            0 |                0 |             0 |               3 |      13 |
| HQ16          |                 2 |            1 |            1 |                0 |             2 |               4 |      10 |
| HQ6           |                 8 |            0 |            0 |                0 |             0 |               1 |       9 |
| S300          |                 2 |            0 |            0 |                1 |             1 |               4 |       8 |
| MLRS          |                 0 |            4 |            0 |                0 |             2 |               2 |       8 |
| HQ22          |                 0 |            1 |            0 |                0 |             0 |               4 |       5 |
| HQ2           |                 3 |            0 |            0 |                0 |             0 |               0 |       3 |
| HQ6A          |                 3 |            0 |            0 |                0 |             0 |               0 |       3 |
| HQ7           |                 1 |            0 |            0 |                0 |             0 |               1 |       2 |

**(b) Radar boxes on camo nets (8) — for analyst verification:** [radar_on_camo_ai.tsv](radar_on_camo_ai.tsv). All 8 sit on objects whose fine label is just `Camo` (no hardware recorded): `1Ec01`, `0Rc04`, `1Oc0L`, `10c02`, `1Rc03`, `1Rc02`, `15c02`, `28c01`. Question for analysts: what is under each net (radar or not)?

## Radar deep dive — 2026-10-14 16:53 IST (Claude)

m0001 deploy predictions, test_80, IoU 0.3, first label only. Radar row per D-20261014-02 (EWS→vehicle no; helicopter boxes not neutral;
stowed rule shown off/on). Script `/sfs/stage2/artifacts/EXP-2026-10-12-01-attr-pattern-merge/radar_deep_dive.py` (env `/sfs/envs/tide`).
Evidence: [radar_outcomes_ai.tsv](radar_outcomes_ai.tsv) (208 radars by instance ID: outcome off/on, best raw radar score, vector rank, reason),
[radar_fp_ai.tsv](radar_fp_ai.tsv) (112 wrong radar boxes), [radar_threshold_sweep_ai.tsv](radar_threshold_sweep_ai.tsv), [radar_deep_dive_ai.json](radar_deep_dive_ai.json).

**Row:** 208 radar GT (176 plain, 30 P_Camo, 2 camo-parent). Rule off: correct 118 · no box 53 · → vehicle 28 · → missile 9 (R 56.7 / P 50.0).
Rule on: correct 133 (15 stowed radars called vehicle become correct; R 63.9 / P 50.0).

**Why the 90 misses (rule off):** detection gap (no radar evidence at any score) **52** · below threshold (raw radar 0.05–0.39) **21** ·
radar ranked 2nd/3rd but not emitted **13** · radar ≥ 0.39 but lost the region in dedup **4**. Lowering the threshold can recover at most ~21.

| family                                                                       | correct / total | main failure                                                                  |
| ---------------------------------------------------------------------------- | --------------: | ----------------------------------------------------------------------------- |
| named modern (HT233, Type780, H200, JY27A, FlapLid, YLC2V/4, 64N6, 96L6E, …) |         69 / 79 | —                                                                             |
| generic `RadarSystem`                                                        |         44 / 90 | 21 detection gap, 12 below threshold, 11 ranked 2nd/3rd; 15 called vehicle    |
| `EWS` (truck-mounted jammer)                                                 |          5 / 19 | 10 called vehicle (detection gap as radar)                                    |
| rare / older (BarLock, SpoonRest, 1L13, PRV16, YLC8B, FanSong, SquatEye, …)  |          0 / 20 | all detection gap — **only 25 training examples across 13 types** (several 0) |

Attributes (correct / total): Position Open 87/133 · Nil 23/58 · Closed 8/17 — Quality 01 65/103 · 02 53/105 — SOC Pixels 91/156 · Site 15/24 ·
Third_Party 12/28 — size < 12 m 25/74 vs ≥ 12 m 93/134 (small radars are the hard ones).

**112 wrong radar boxes:** 66 on no object · 16 on trucks (TransportVehicle 12, EngineeringVehicle 4) · 11 on missiles · 8 on camo nets ·
5 on helicopters · 6 other. The 66 no-object boxes: score quartiles 0.47 / 0.59 / 0.80 (17 ≥ 0.8); 14 in scenes with no radar at all;
nearest radar median 106 m (26 within 100 m = clutter inside radar sites); median box length 13 m.

**Radar threshold sweep** (rule off → on; current 0.39 ≈ the 0.40 row):

| thr | R off | P | F1 off | R on | F1 on |
|--:|--:|--:|--:|--:|--:|
| 0.40 | 56.7 | 50.7 | 53.5 | 63.9 | 56.6 |
| 0.50 | 53.8 | 55.8 | 54.8 | 62.5 | 59.0 |
| 0.65 | 51.4 | 63.5 | **56.8** | 60.6 | 62.0 |
| 0.90 | 43.3 | 79.8 | 56.1 | 52.9 | **63.6** |

Readings: (1) radar's misses are mostly **detection gaps** (generic RadarSystem, EWS, rare types) — a training-data problem, not a rule problem;
(2) its precision problem is mostly **boxes on nothing** at mid scores, so a higher radar threshold (≈0.65 off / 0.90 on) trades a little recall
for a lot of precision; (3) named modern radars are already ~87 % correct.

### Stowed radars (Position Closed/Nil, 74) — 19:53 IST (Claude, researcher question)

[radar_stowed74_ai.tsv](radar_stowed74_ai.tsv) lists all 74 by instance ID. Outcomes: **31 correct as radar** (Closed 8, Nil 23) · **15 correct only
via the stowed rule** (called vehicle) · **23 no box** · **5 called missile**. Of the 28 failures: 19 detection gap (no radar evidence at any score),
6 below the 0.39 threshold (raw 0.06–0.29), 3 radar ranked 2nd/3rd in the box's scores, 1 radar 0.57 lost its region to a missile box.
By type: RadarSystem 18 of 28 failures; EWS 3; rare types (1L13, YLC8B, JY8) 5; H200 1; HT233 1. Most no-box cases have `vehicle` or
`logistic` as the box's top class — the model sees a truck-shaped object but not a radar.
