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
- Mixed floors (researcher, ~15:20 IST): vehicle / smallvehicle / camo P ≥ 50 %, others P ≥ 40 % (`PMIN=0.40 PMIN_CLASS=vehicle:0.5,smallvehicle:0.5,camo:0.5`,
  `reports/v3_y11l_P50v_P40.md`). IoU 0.3 MICRO P 49.3 / R 77.9 / F1 60.3, MACRO(7) 43.6: vehicle 50.2/83.5 (conf 0.20) · smallvehicle 50.3/74.1 (0.405) ·
  camo 50.1/76.7 (0.45) · missile 40.8/53.3 · radar 40.6/62.1 · tank / arty as before. IoU 0.5: P 49.2 / R 73.9.
- Tank diagnosis (`code/tank_diag_remap.py`, v3 y11l): remapped tank GT 163 = SPATracked 57 + Tank 55 + SPAWheeled 35 + K9 16.
  Found at tank conf 0.3 / 0.7 / 0.96: Tank 80 / 73 / 55 % · SPATracked 25 / 19 / 18 % (21 low-conf other class, 12–13 → vehicle) · SPAWheeled 40 / 14 / 3 %
  (→ vehicle 6–10, → missile 3, tank box at low conf) · K9 0 % (14 / 16 no box; one scene). Recall failure = SPA, not tanks.
  Tank boxes (precision 10 / 12.5 / 19 %) land on: at 0.96 — ICV now vehicle 60 + ICV labelled camo 40 (100 / 219), transport vehicle 38, background 31
  (22 in 25JUN16072810), real Tank 30, SPATracked 10; at 0.3 — transport vehicle 170, background 145, ICV 99 + 93, camo SPATracked 34, logistics 29, LTV 29.
  Cause: training tank_spa held ~1,650 ICVs (rule 3 now maps ICV → vehicle); without the ICV boxes precision at 0.96 would still be only ~34 %.
- ICV destinations (`code/icv_diag_remap.py`, v3 y11l, F1-opt thresholds; recall-heavy P50/P40 in brackets): 585 test ICVs.
  Camo-labelled (PC05, target camo) 368: correct camo 78.3 % [84.5], → vehicle 9.2 % [11.4], → tank 7.1 % [3.8]. Tank-labelled non-P_Camo (target vehicle) 185:
  correct vehicle 31.4 % [42.2], → camo 20.5 % [19.5], → tank 18.4 % [17.8], no box 12.4 % (19 in 22JUL07045805). Tank-labelled P_Camo=Yes (vehicle, camo ok) 32:
  → tank 50 %, camo 37.5 %. Scene 23JUL02025233 holds 389 ICVs: 289 camo-labelled (270 found as camo) and 100 tank-labelled (→ tank 49, → camo 30, camo-ok 12)
  — the same ICVs in one scene carry two labels, which the remap turns into camo vs vehicle. 24APR06053032: 69 camo-labelled ICVs mostly → vehicle (44).
- Tank without the SPA merge (`code/tank_nospa.py`, researcher ~16:00 IST): tank GT = 55 real Tanks (none camo-conditional). v3 y11l tank boxes:
  recall at conf 0.97 (the P50/P40 table point) 54.5 % (30/55) vs 23 % with SPA merged; conf 0.9 65.5 · 0.7 72.7 · 0.3 80.0 · 0.05 92.7 %.
  Precision 15.2 % (SPA = separate class, tank boxes on SPA = FP) / 15.8 % (SPA ignored) at 0.97; ≤ 16 % at every conf → P ≥ 40 unreachable
  (remaining FPs = ICVs, transport vehicles, background).
- Tank precision (researcher ~16:10 IST): at conf 0.97 (198 tank boxes, 38 correct) false tank boxes land on ICV now vehicle 59 + ICV labelled camo 35
  (94 = 47 % of all boxes; 84 of them in 23JUL02025233), transport vehicles 31 (16 in 25SEP14032108, 14 in 23JUL02025233), background 28
  (20 in 25JUN16072810), camo SPATracked 3, logistics 2. 23JUL02025233 alone gives ~100 of the 160 false tank boxes. At 0.7: transport vehicles 96,
  ICV 91 + 69, background 77, camo SPATracked 21, LTV 16.
- ICV → tank variant (researcher ~16:15 IST; `remap.py` env ICV_TO=tank = rule 3 to tank, PC05 ICVs stay camo; SPA_MERGE=0 = SPA not scored), same P50/P40
  floors (`reports/v3_y11l_P50v_P40__ICVtank_{noSPA,withSPA}.md`). Tank (Tank + 217 ICV, no SPA, GT 268): P 42.7 / R 33.6 at conf 0.965 (floor met; F1-opt
  P 38.7 / R 39.3). Tank + ICV + SPA (GT 378): P 40.3 / R 32.0 at 0.90 (F1-opt 37.2 / 35.9). Doc rules: P 19.2 / R 23.3. Vehicle R 83.5 → 84.9 at P 50;
  MICRO 49.3 / 77.9 → 49.5 / 78.1. Found as tank at 0.965: Tank 55 %, ICV 28 %, SPATracked 16 %, SPAWheeled 3 %, K9 0 %; remaining false tank boxes:
  camo-labelled ICV 39, transport vehicles 34, background 30.
