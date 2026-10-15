# Weekly update — missile & radar detection (week of 12 Oct 2026) — AI-authored draft

Author: AI agent (Claude), drafted 2026-10-15 for the researcher to edit before sending. Human companion: [manager_report.md](manager_report.md).
Evidence: [week review](review_ai.md), experiment records in [experiments/](experiments/README_ai.md).

---

This week I looked closely at where our object detector fails on **missiles and radars** in satellite images, using the
extra details the analysts record for every object (camouflage, whether a launcher is loaded, whether a radar antenna is
folded, etc.). I found clear, visually consistent patterns, turned them into scoring rules that match how an analyst would
judge the output, and ran ~20 retraining/evaluation experiments to test them. All numbers below are on our fixed test set
of 80 images. *Recall* = share of real objects we find; *precision* = share of our alarms that are correct.

## Missiles

**What I found**
- **Our official score undercounts the model.** 145 partly camouflaged missiles were counted as "camouflage" in the official
  labels, so every correct "missile" call on them was marked wrong. Scoring them as missiles (with camouflage also accepted):
  recall **41% → 67%**, precision **30% → 72%** — same model, fairer judgement.
- **The main real error is missile vs. truck.** Empty launchers and transport/reload vehicles look like ordinary trucks; MLRS
  (rocket launchers on a truck chassis) are called "vehicle" 32 of 48 times.
- **46 missiles are never detected at all** (mostly HQ9, HQ12, HQ6 and generic "MissileSystem").
- **Most wrong missile alarms sit on large trucks inside missile sites** (39 of 62) — likely support/reload vehicles that sit on
  the boundary of how they are labelled. Flagged for analyst review.

**What I tried:** retraining with empty launchers and transport vehicles labelled as "vehicle" (only loaded launchers stay "missile").

| model | overall score* | missiles found / alarms correct | radar score* |
|---|--:|--:|--:|
| current deployed model (two networks combined) | 56.6 | 67% / 72% | 53.2 |
| larger network alone (no retraining) | **57.8** | 61% / 83% | **55.9** |
| larger network retrained with the missile relabelling | 56.7 | 60% / **89%** | 52.2 |
| same, gentler retraining | 57.2 | 56% / **92%** | 55.0 |

\*F1 score, 0–100, balancing found vs. correct.

**What worked:** the relabelling makes missile alarms much more trustworthy (precision 72% → 89–92%) and gives the best
missile score so far. **Honest caveat:** the gain over the un-retrained larger network is modest, and recall drops a little.

## Radars

**What I found**
- **Modern, named radar types are mostly solved:** 87% found.
- **Misses are mainly radars the model never sees as radar** (52 of 90 misses): objects only labelled generically as
  "RadarSystem" (44 of 90 found) and **older/rare types — 0 of 20 found**, because training has only 25 examples spread over 13 types.
- **Wrong radar alarms are mostly on empty ground** (66 of 112). Raising the radar confidence cut-off from 0.39 to ~0.65
  improves precision 51% → 64% for a small loss in recall (overall radar score +3).
- **Folded (stowed) radars look like trucks.** Retraining with them labelled "vehicle" made radar alarms more precise (59% → 64%).

## Rules I set for fair scoring

**Missiles** (analyst-agreed, applied to the model's first answer only)
- Partly camouflaged missile: "missile" or "camouflage" both count as correct.
- Camouflage net over missile equipment: "missile" or "camouflage" both correct.
- Empty or unrecorded launcher (no canisters): "missile" or "vehicle" both correct. Loaded launcher must be "missile".
- Rejected: radar ↔ missile swaps, transport vehicles as "vehicle", loaded camouflaged launchers as "vehicle" (kept for visual check).

**Radars**
- Camouflaged radar: "radar" or "camouflage" correct.
- Electronic-warfare trucks (EWS) must be called radar; radar alarms on helicopters count as errors.
- Folded radar accepted as "vehicle": tested both ways — recall 57% → 64% when accepted. *Decision pending.*

## Next steps
1. **Data, not tuning, is the bottleneck for the remaining misses:** request more examples of rare radar types, EWS and MLRS,
   and ask analysts to record the specific radar type instead of the generic "RadarSystem".
2. **Analyst review** of the flagged cases: trucks inside missile sites, radar alarms on camouflage nets, missiles that are never found.
3. **Pick the next model fairly:** re-tune the confidence cut-offs for the top candidates (the larger network alone and its
   relabelled versions), then confirm on the **new 26-image delivery (15 Oct)** so the choice is not tuned to one test set.
4. Finalise the folded-radar rule and adopt the higher radar cut-off if the reduced recall is acceptable.

Also this week: set up a single, synced research workspace where every experiment, result and decision is recorded.
