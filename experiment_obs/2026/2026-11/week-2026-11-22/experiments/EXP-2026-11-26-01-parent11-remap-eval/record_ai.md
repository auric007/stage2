# EXP-2026-11-26-01-parent11-remap-eval — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-26_ai.md](../../daily/2026-11-26_ai.md).
Models scored (existing predictions, no inference): [EXP-2026-11-23-01](../EXP-2026-11-23-01-lowres-poc-train/record_ai.md) v1, [EXP-2026-11-24-01](../EXP-2026-11-24-01-lowres-poc-v2/record_ai.md) v2,
[EXP-2026-11-25-01](../EXP-2026-11-25-01-lowres-poc-v3/record_ai.md) v3 (y11l epoch 26).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-26-01-parent11-remap-eval`

Status: **completed** 2026-11-26.

## Request summary and authorization
- Researcher, 2026-11-26: metrics of the earlier best model on the 136-scene lowres_poc test under the owner-approved remapping rules in
  `/sfs/stage2/14_PARENT11_ONEOFF_INSTANCE_REMAPPING_20261126.md`; two operating points: recall-heavy (read as: precision ≥ 30 %, max recall) and F1-optimal.

## Setup
- GT remap `code/remap.py` (rules 1–8 + conditional camo for non-PC05 P_Camo=Yes). Audit reproduces the doc exactly: PC05 761, conditional 249,
  EWS → vehicle 36, ICV → vehicle 217, logistic → vehicle 690, loaded launcher-family missile 244, other PC02/PC07 → vehicle 63, SPA → tank 108.
  Final GT: vehicle 5,142 · smallvehicle 1,436 · camo 761 · missile 244 · radar 203 · tank 163 · aerial 107 · arty 65 (radome / concealment not scored).
- Scorer `code/score_remap.py`: predictions logistic → vehicle, spa / tank_spa → tank, merged-class duplicates (IoU ≥ 0.4) keep max (detectors give one class per
  box with sigmoid scores — the doc's probability sum before argmax has no direct equivalent); Delete by prediction centre; greedy rotated IoU 0.3 (and 0.5);
  conditional objects: primary hit → primary TP (camo box ignored), else camo hit → camo TP and no primary FN, else primary FN. Per-class thresholds on a
  0.005 grid by coordinate ascent; F1 mode = per-class best F1; RECALL mode = per-class max recall with P ≥ 30 % (max-precision point if unreachable).
  aerial never trained → reported, kept out of MICRO / MACRO(7).

## Results (IoU 0.3) — `reports/<set>.md|json`
| set | F1-opt MICRO P/R/F1 | MACRO(7) | recall-heavy MICRO P/R/F1 | MACRO(7) |
|---|---|--:|---|--:|
| v1 y11l | 66.7 / 64.8 / 65.7 | 48.0 | 35.2 / 82.0 / 49.2 | 35.7 |
| v1 ens | 64.4 / 64.7 / 64.6 | 47.9 | 29.9 / 84.5 / 44.2 | 34.4 |
| v2 y8m | 65.8 / 64.2 / 65.0 | 48.6 | 33.8 / 81.0 / 47.7 | 33.5 |
| v2 y11l | 67.6 / 63.9 / 65.7 | 51.2 | 32.8 / 83.0 / 47.0 | 38.3 |
| v2 ens | 66.9 / 65.5 / 66.2 | 50.8 | 30.2 / 84.8 / 44.5 | 36.4 |
| **v3 y11l** | **70.6 / 64.6 / 67.4** | **51.3** | 33.6 / 83.7 / 47.9 | 35.8 |

v3 y11l per class, F1-opt (P/R/F1): vehicle 76.9/67.4/71.8 · smallvehicle 67.6/63.5/65.5 · camo 71.0/68.9/70.0 · missile 61.1/42.0/49.8 · tank 18.7/25.2/21.5 ·
radar 81.8/49.5/61.7 · arty 12.1/41.5/18.7 · aerial 0. Recall-heavy: vehicle 35.4/88.4 · smallvehicle 30.7/83.0 · camo 30.1/85.5 · missile 30.4/60.4 (conf 0.47) ·
radar 32.8/64.9 (conf 0.335) · tank P ≥ 30 unreachable (max 19 %) · arty P ≥ 30 only at R 3 %. IoU 0.5: F1-opt MICRO 65.5 / MACRO(7) 49.4; recall-heavy 32.5 / 79.8.
- Tank precision is capped by the mapping: of 178 tank FPs at conf 0.96, 60 are tank-labelled ICVs now vehicle (R3), 40 camo-labelled ICVs, 38 transport vehicles,
  31 background — the models were trained with ICV as tank. Missile FPs: mostly transport / engineering vehicles; empty launchers (R5) only 7. Radar: EWS → vehicle 3.

### Follow-up (2026-11-26 ~15:10 IST): 40 % operating points, v3 y11l — `reports/v3_y11l_P40.md|json`
- `code/score_remap.py` was found **deleted** from `code/` (code dir modified 14:28 IST; no copy elsewhere under /sfs/stage2). Recreated from the
  AI-authored version with the same logic + `PMIN` / `RMIN` env options; re-scoring v3 y11l reproduces the original `reports/v3_y11l.json` exactly (TP/FP/n/conf, all classes, both IoUs).
- Precision ≥ 40 % (max recall), IoU 0.3: MICRO P 39.9 / R 81.7 / F1 53.6, MACRO(7) 40.7. vehicle 40.1/87.1 · smallvehicle 40.1/79.7 · camo 40.1/81.8 ·
  missile 40.8/54.2 (conf 0.73) · radar 40.6/62.1 (conf 0.465) · tank unreachable (max P 19) · arty P ≥ 40 only at R 3. IoU 0.5: P 40.1 / R 77.6.
- Recall ≥ 40 % (max precision), IoU 0.3: MICRO P 74.4 / R 40.9; vehicle 93.5 · smallvehicle 82.2 · camo 92.1 · missile 62.5 · radar 87.2 · tank 10.6 · arty 12.1.
