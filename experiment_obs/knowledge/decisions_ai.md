# stage2 decisions — AI-authored

Author: AI agent (Claude). Human/legacy companion: [decisions.md](decisions.md).
Moved here on 2026-10-14 from `decisions.md` under the human/AI authorship rule (content unchanged except links).

## D-20260902-01 - Deployment operating point for m0001

Date: 2026-09-02 (recorded 2026-10-14). Decision: per-class deploy thresholds radar 0.39, missile 0.85, arty 0.86, logistic 0.77,
camo 0.42, smallvehicle 0.24, tank 0.54, vehicle 0.84; cross-class dedup IoU 0.4 with priority radar, missile, arty, logistic.
Result P 53.6 / R 60.0 on test_80 (protocol P01). Evidence: [EXP-2026-09-01-01](../2026/2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record_ai.md).
Revisit if a new model or rule set changes per-class precision.

## D-20261009-01 - Protocol P02 (camo credit)

Date: 2026-10-09. A camouflaged object (labels_detailed `camo` with hardware fine label, or P_Camo = Yes) counts as correct if predicted
`camo` or its hardware class, in the hardware row. Evidence: [EXP-2026-10-09-01](../2026/2026-10/week-2026-10-04/experiments/EXP-2026-10-09-01-detailed-camo-credit/record_ai.md).
Partly superseded for missile by D-20261013-02.

## D-20261013-02 - Protocol P03 missile row (user rule set, first label only)

Superseded for reporting by D-20261023-01 (2026-10-23).

Date: 2026-10-13. Missile GT = labels_detailed missile + camo-parent missile hardware (319). Correct if: `missile`; P_Camo → also `camo`;
camo-parent missile hardware → also `camo`; Is_Empty(ML) Yes or Nil (canisters not loaded) → also `vehicle`; loaded (No) → `vehicle` NOT accepted.
Rejected: P_Camo-loaded → vehicle (flagged for visual check), Transporter/Transloader → vehicle (model only partly consistent), radar ↔ missile
(distinct classes). Second labels and score relabel deferred. Result recall 67.1 % (214/319), precision 72.1 % (160/222).
Evidence: [EXP-2026-10-12-01](../2026/2026-10/week-2026-10-11/experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md) and the user's
[Missiles Rules](../2026/2026-10/week-2026-10-11/Missiles%20Rules.md). Revisit after the visual checks.

## D-20261014-01 - Notebook is the single tracking location for stage2

Date: 2026-10-14. All writing, tracking and result analysis goes into experiment_obs (synced). Research code, weights, predictions and other
large outputs stay outside in `/sfs/stage2/artifacts/<EXP-ID>/`, `models/`, `data/`; the user moves any script into the notebook if it should be
published. The earlier `/sfs/stage2/experiments`, `journal/` and `registry/` were migrated here and removed; earlier IDs e0001–e0007 are kept as
cross-references in each record. Evidence: [daily 2026-10-14](../2026/2026-10/week-2026-10-11/daily/2026-10-14_ai.md).

## D-20261014-02 - Protocol P03 radar row (partial, researcher decisions 2026-10-14 16:10 IST)

Superseded for reporting by D-20261023-01 (2026-10-23).

- **EWS → vehicle: rejected.** EWS stays radar-only (10 of 19 EWS currently predicted vehicle remain errors).
- **Radar boxes on helicopters (aerial GT): not neutral** — they stay false positives.
- **Stowed radar (Position Closed/Nil) → vehicle:** undecided — researcher asked for one experiment with the rule on and one with it off.
- Base row (unchanged): radar GT = labels_detailed radar + camo-parent radar hardware (208); accept radar, + camo if P_Camo / camo parent; first label only.
- Reference (m0001 deploy, Sept pipeline): rule off R 56.7 / P 50.0 (F1 53.2); stowed→vehicle on R 63.9 / P 50.0 (F1 56.1).
Evidence: EXP-2026-10-12-01 radar_rule_v1.py; [daily 2026-10-14 AI log](../2026/2026-10/week-2026-10-11/daily/2026-10-14_ai.md).

## D-20261023-01 - P03 dropped from reporting; P04 single-class attribute remap (draft)

Date: 2026-10-23 19:05 IST. Researcher: "remove P03 and tell me best metrics; you can map any attribute to any class if it is boosting
the recall, but only one class". P03 (several accepted classes per object, D-20261013-02 / D-20261014-02) is no longer used for reported
numbers; those entries stay as history. New draft **P04**: official P01 scoring with each object remapped to exactly one class by 14
attribute rules (camouflaged HQ9/S300/HQ12/MissileSystem → missile; MLRS, plain HQ9, plain MissileSystem, camouflaged HQ16, EWS, camo-parent
LTV, ABLV → vehicle; SPATracked and camouflaged tanks → tank). Rules were chosen on test_80 m0001 evidence (optimistic) and await
the researcher's review. Best-per-class MICRO F1 63.5 / MACRO 54.4 (P01: 60.2 / 49.6); m0001 alone 63.0 / 53.8.
Evidence: [EXP-2026-10-23-02](../2026/2026-10/week-2026-10-18/experiments/EXP-2026-10-23-02-single-class-attr-remap/record_ai.md).

## D-20261026-01 - Camouflaged missiles and radars go to their hardware class; P04 v2 (draft)

Date: 2026-10-26 16:35 IST. One-class-per-object comparison on 21 models (EXP-2026-10-26-01).
- **Camouflaged missiles (157) → missile.** The official "→ camo" is the worst option for missile (F1 36.4 vs 59.4–62.1) and for camo (46.5 vs 49.5).
  Best overall: the P04 fine-type split (HQ9/S300/HQ12/MissileSystem → missile, MLRS/HQ16 → vehicle).
- **Camouflaged radars (30, P_Camo) → radar.** Added to P04 as rule 15 (radar F1 53.3 → 58.8 best per class, MACRO +0.6, recall +0.3).
- **Why it works:** every model was trained with these objects as their hardware class (labels_detailed), so the official camo mapping penalized correct predictions.
- **P04 v2 result:** best per class MICRO 63.6 / MACRO 55.0; m0001 alone 63.0 / 54.0. Chosen on test_80 (upper bound); pending the researcher's review.
- Evidence: [EXP-2026-10-26-01](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-01-pcamo-missile-radar-mapping/record_ai.md).
