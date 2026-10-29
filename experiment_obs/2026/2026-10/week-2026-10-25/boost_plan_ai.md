# Boosting the deployment selection — what worked per class, the requirement, and the experiment plan (AI-authored, 2026-10-29)

Author: AI agent (Claude), 2026-10-29 ~21:50 IST. Human companion: [boost_plan.md](boost_plan.md).
Correlation: EXP-2026-10-28-02 · deployment selection in [knowledge/best_combination_ai.md §2](../../../knowledge/best_combination_ai.md).
**Planning only.** Nothing here is approved or started.

All numbers below are on Test116 (116 images), IoU 0.3, labels_simple, with the 25 prediction sets of EXP-2026-10-28-02.

## 1. The requirement, read from the researcher's picks

Each pick is the **highest recall that still keeps precision above a per-class floor**. The exceptions are camo and logistic, where the pick is the best F1.

| class | picked | rule that reproduces it exactly |
|---|---|---|
| radar | new_ens_last @0.5 (alternatives 0.4 / 0.3) | R at P ≥ 50 (P ≥ 45 / P ≥ 40) |
| missile | EXP-15 ens @0.7 (alternative EXP-09 ens @0.6) | R at P ≥ 50 (P ≥ 45) |
| arty | new_ens_last @0.8 | R at P ≥ 45 |
| camo | new_ens @0.9 | best F1 |
| logistic | new y11l @0.5 | best F1. P never gets above ~35 at useful recall. |
| smallvehicle | EXP-18 @0.4 (alternative new_ens_last @0.4) | R at P ≥ 60 = best F1 |
| tank | new_ens @0.8 (alternative new y8m @0.6) | R at P ≥ 65 |
| vehicle | EXP-07 ens @0.9 (alternative EXP-14 ens @0.8) | best F1 / R at P ≥ 64 |

**Proposed objective for all future work:** maximise recall at the floors P ≥ 50 for radar, missile and camo; P ≥ 45 for arty; P ≥ 60 for smallvehicle, tank and vehicle; best F1 for logistic. Measure it **after the cross-class merge**. *(To be confirmed by the researcher.)*

Two cases the floors reveal:
- **smallvehicle at P ≥ 50:** EXP-18 @0.2 gives P 50.7 / R 70.9. That is the same recall as the new_ens_last row (P 49.1 / R 71.9) with higher precision and one model instead of two.
- **tank, if the floor is 60:** new_ens @0.7 gives P 61.6 / R 85.1.

## 2. What worked, and why

Recall at the floor for each set:

| set | arty@45 | camo@50 | logistic@30 | missile@50 | radar@50 | smallvehicle@60 | tank@60 | vehicle@60 |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| m0001 y8m alone | 45.8 | 49.3 | 7.7 | 56.7 | 51.6 | 51.8 | 76.6 | 78.7 |
| m0001 y11l alone | 19.7 | 50.4 | 20.8 | 53.8 | 59.3 | 58.6 | 82.6 | 81.0 |
| **m0001 ensemble** | 44.6 | 54.7 | 24.5 | 58.6 | 61.5 | 60.1 | 82.2 | 82.8 |
| new y8m best | 49.4 | 52.3 | 18.8 | 54.2 | 56.0 | 54.8 | 82.6 | 80.4 |
| new y11l best | 42.6 | 55.2 | 27.2 | 53.5 | 58.5 | 50.5 | 78.0 | 79.1 |
| **new ensemble, best.pt** | 51.4 | 55.7 | 25.2 | 55.7 | 61.5 | 58.9 | **85.1** | 81.6 |
| **new ensemble, last.pt** | **53.0** | **58.2** | 18.8 | 55.7 | **63.6** | 59.3 | 79.1 | **84.3** |
| m0001 y8m + 10-epoch fine-tune (EXP-04) | 41.0 | 46.1 | 22.8 | 59.1 | 53.8 | 48.1 | 78.0 | 81.5 |
| m0001 y11l + 10-epoch fine-tune (EXP-18) | 19.3 | 53.4 | 12.8 | 54.2 | 53.8 | **63.0** | 79.1 | 81.1 |
| ens: y8m fine-tune + y11l (EXP-09, control labels) | 45.0 | — | 25.5 | 59.6 | 58.9 | 59.0 | 79.1 | 83.0 |
| same, relabel R (EXP-15) | 45.8 | — | 24.8 | **59.8** | 60.0 | 59.7 | 80.0 | 83.4 |
| same, relabel C (EXP-07) | 44.6 | — | 24.8 | 58.1 | 59.6 | 59.5 | 81.1 | 83.4 |
| same, relabel D (EXP-08) | 43.4 | — | 24.8 | 58.4 | 58.9 | 59.5 | 81.2 | 83.2 |

"—" means precision never reaches the floor.

1. **Ensembling two different architectures is the most reliable gain.** At the same precision, y8m + y11l adds about 3 to 8 recall over the better single model:
   - radar 59.3 → 61.5–63.6;
   - camo 50.4 → 54.7–58.2;
   - missile 56.7 → 58.6.
   Every pick except logistic and smallvehicle is an ensemble.
2. **More training data (Train359) helps arty, tank and camo**: new ens best vs m0001 ens, arty +6.8, tank +2.9, camo +1.0. It does **not** help missile (−2.9), smallvehicle (−1.2) or vehicle (−1.2). These single-seed differences are close to noise.
3. **More training at the end raises recall** for radar (+2.1), camo (+2.5), vehicle (+2.7) and arty (+1.6). This compares last.pt (epoch 67–69) with best.pt (epoch 37–39), which early stopping chose by mAP50-95 fitness on test tiles. Tank (−6.0) and logistic (−6.4) go the other way.
   - Conclusion: one checkpoint does not suit every class. The fitness-chosen best.pt is too early for the rare classes.
4. **The "m0001 + 10-epoch low-learning-rate fine-tune" lineage still holds missile and smallvehicle.** That is EXP-04, EXP-18 and the EXP-09/15/07 ensembles.
   - The fine-tune alone adds missile +2.4 (y8m) and smallvehicle +4.4 (y11l).
   - The new recipe has never had this extra fine-tune stage.
5. **The relabels themselves (R, C, D) do not matter.** All of them sit within about ±1.5 of the control fine-tune (EXP-09). EXP-15 and EXP-07 win because they are fine-tuned ensembles, not because of their labels.
6. **No leak.** The old fine-tunes trained on `data/derived/train_final`, which contains no Test116 scene. However, they chose best.pt on 77 Test116 scenes, as all models so far did.

**Cost of the cross-class merge** (default preset, recall before → after):
- tank 82.3 → 70.4;
- vehicle 76.4 → 70.0;
- camo 55.7 → 51.6;
- missile 59.8 → 56.2;
- smallvehicle 63.0 → 59.7.

Scores from different sets are compared directly, even though each class has its own threshold. A vehicle box at 0.91 therefore beats a tank box at 0.85, although the tank box is further above its own threshold (0.8).

## 3. Plan

### Phase 0 — CPU only, no GPU, ~2–3 h. Can run now.
- **E0a, honest estimate.** Split Test116 into two halves of sites. Choose sets and confidences on one half under the §1 objective, score the other half, then swap. This tells us how much of 65.0 / 57.3 comes from choosing on the test.
- **E0b, merge rule.** Compare boxes by their margin above their own class threshold, (score − thr) / (1 − thr), instead of by raw score. Try per-class-pair IoU for the confusable groups: tank / vehicle / smallvehicle / camo.
  - The target is the merge recall loss above, most of all tank (−12) and vehicle (−6).
  - Choose on one half, check on the other.
  - Expected: recover a few recall points on tank and vehicle. It may also lose a little precision.
- **E0c, cross-set fusion of saved predictions.** For example, new ensemble + m0001 ensemble for radar and missile.
  - This only approximates true 4-model fusion, because the saved boxes are already fused per set. If it helps, we redo it properly in Phase 2.

### Phase 1 — data decision (researcher)
- **Option A:** keep Train359, choose on test with E0a cross-fitting.
- **Option B:** move the 71 images that the subset loop showed can leave Train into a VAL split and train on 288.
  - Then best.pt, sets and confidences are all chosen on VAL, and Test116 is scored once.
  - VAL holds only about 53 radar and 78 missile objects, so per-class choices on it will be noisy.
  - This needs approval to change the /sfs/data split.

### Phase 2 — GPU, about 1.4–1.8 days (one L40S)

| run | what | GPU |
|---|---|--:|
| G1 | New recipe, seeds 1 and 2, y8m + y11l. **No early stopping (100 epochs), checkpoint every 10 epochs.** This gives the per-class epoch curve and allows a per-class checkpoint choice. | ~35 h (17.4 h per seed: y8m 7.0 + y11l 10.4) |
| G2 | **Fine-tune stage on the new models**: 10 epochs, lr 2e-4, the stage that won missile and smallvehicle for m0001. Applied to seed 0's last.pt for both architectures, then to the G1 seeds. | ~1.75 h per pair → ~5 h |
| G3 | TTA inference for each new set, ensemble of seeds (4–6 models), then re-score and re-select under the §1 objective after the merge. | ~3–4 h |
| — | Relabels R, C and D | **skipped**: within noise (point 5 above) |

Cheaper version: keep patience 30 as before (~25 h for 2 seeds) and skip the per-epoch curve. That totals ~1.4 days.

### Expected gains (estimates, not measurements)

| source | where | expected |
|---|---|---|
| merge rule (E0b) | tank, vehicle, camo after merge | +2 to +6 R on tank and vehicle, small on the others |
| seed ensembles (G1 + G3) | all classes, mostly radar, arty and camo | +1 to +3 R at the same floor (more members, smaller step than the y8m + y11l pair gave) |
| fine-tune stage on new models (G2) | missile, smallvehicle | +2 to +4 R, judging by +2.4 / +4.4 on m0001 |
| per-class checkpoint (G1 curve) | radar, arty, camo vs tank, logistic | +1 to +2 R where last.pt and best.pt disagree |
| honest selection (E0a / VAL) | reported number | **likely −0.5 to −2**; this is what the noise question is about |

Gains from different sources usually do not add up fully. The realistic aim is radar and missile at R 66–68 with P ≥ 50, tank and vehicle recovering about 4–6 recall after the merge, and seed noise finally known. A light option (at most 2 models) is reported next to the best mix each time.
