# stage2 findings — AI-authored

Author: AI agent (Claude). Human/legacy companion: [findings.md](findings.md).
Moved here on 2026-10-14 from `findings.md` under the human/AI authorship rule (content unchanged except links).

## F-20261002-01 - Stage-2 errors are mostly classification, not detection

Status: current. On test_80 (m0001, IoU 0.3) 93.4 % of objects get a box; the true class is in the box's top-3 for 87.1 % vs top-1 69.8 %.
At deploy thresholds 1,123 of 2,041 misses got a box of the wrong class, usually `vehicle`.
Evidence: [EXP-2026-09-01-01](../2026/2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record_ai.md),
[EXP-2026-10-08-01](../2026/2026-10/week-2026-10-04/experiments/EXP-2026-10-08-01-imagewise-deploy/record_ai.md).

## F-20261013-01 - Missile false positives sit on large trucks inside missile sites

Status: current, pending visual check. 39 of 62 wrong missile boxes are on TransportVehicle/EngineeringVehicle labels; they are 10–18 m long,
22 of 40 within 50 m of a labelled missile, and the model scores them ≥ 0.9 vehicle as well. Likely a labelling boundary for battery support
trucks. Evidence: EXP-2026-10-12-01 attachment `missile_fp_on_vehicles.tsv`.

## F-20261014-01 - Radar's weak side is precision

Status: current. Under P03-style rules radar precision is 50.0 % (112/224); 66 radar boxes sit on no object. Named radar types are ~88 % correct,
generic `RadarSystem` 44/90, `EWS` 5/19 (10 called vehicle), rare older types ~0/18. Evidence: EXP-2026-10-12-01 record.
