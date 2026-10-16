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

## F-20261016-01 - Fine-tuning m0001's members for 10 epochs costs overall F1 and radar, even on unchanged labels

Status: current. y8m: deploy F1 57.1 → 55.0, missile precision 78.2 → 65.1 (EXP-2026-10-14-05 → -04). y11l: 57.8 → 57.2, radar F1 55.9 → 50.9
(EXP-2026-10-14-17 → -18). Lower LR (5e-5) reduces the drift (EXP-2026-10-15-02). Conditions: V100, 10 epochs, AdamW, m0001 thresholds, test_80.

## F-20261016-02 - Missile (D) and radar (R) relabels raise precision relative to their controls

Status: current. D: missile precision +12.1 (y8m, EXP-03 vs -04), +10.5 (y11l, EXP-13 vs -18). R: radar precision +4.7 (y8m, EXP-12 vs -04),
+8.1 (y11l D+R, EXP-19 vs -18). Recall gains are small or negative; combining D+R in one y11l lost missile recall (F1 66.8).

## F-20261016-03 - The original YOLO11l alone beats the m0001 ensemble at m0001's thresholds

Status: current, threshold caveat. Deploy F1 57.8 vs 56.6, radar F1 55.9/59.8 vs 53.2/56.1, missile F1 70.1 vs 69.5 (EXP-2026-10-14-17 vs -10).
Every new ensemble lost precision at these thresholds. Caveat: thresholds were tuned for the m0001 ensemble on test_80 itself; a fair model
choice needs per-model thresholds on a separate validation split.
