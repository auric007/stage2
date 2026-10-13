### Fixed setup (unchanged from the official metric)

1. **Model:** the sep_w1 final ensemble (YOLOv8m-OBB + YOLO11l-OBB, TTA, noisy-or).
2. **Deployment thresholds:** missile 0.85 · radar 0.39 · arty 0.86 · logistic 0.77 · camo 0.42 · smallvehicle 0.24 · tank 0.54 · vehicle 0.84. Overlapping boxes (IoU ≥ 0.4) are de-duplicated, with priority to radar, missile, arty and logistic, and boxes in delete regions are removed.
3. **Test set:** 80 images (Test1 43 + Test2 37). A box matches an object at **IoU ≥ 0.3**.
4. **First label only.** Each deployed box counts with its single predicted class. No second labels, no top-2/top-3 and no score-based relabelling.
5. **Matching:** boxes in score order, each matched to the best-overlapping unmatched object it's acceptable for. One box ↔ one object.

### Which objects are in the missile row (GT = 319)

6. **Missile GT comes from `labels_detailed`, not the simple mapping.**
    - All 307 objects with class `missile`, including the 145 with P_Camo = Yes, which the simple mapping had moved to camo.
    - Plus the **12 camo-parent objects whose fine label is missile hardware**: HQ9 6, MissileSystem 5, HQ22 1.

### What counts as correct for a missile object (recall)

7. **Default:** predicted `missile`.
8. **R1, P_Camo = Yes:** predicted `missile` or `camo`.
9. **R2, camo parent with a missile fine label:** predicted `missile` or `camo`.
10. **R3, Is_Empty(ML) = Yes or Nil** (canisters not loaded, so it looks like a truck): predicted `missile` or `vehicle`. This applies to every missile system, MLRS included.
11. **Is_Empty = No (canisters loaded):** `vehicle` is not accepted. Only `missile`, or `camo` if R1 applies.
12. **R1 and R3 combine:** a P_Camo missile that's also empty accepts `missile`, `camo` or `vehicle`.

### Precision

13. **Missile precision = missile boxes on an accepted missile object ÷ all 222 missile boxes.** These rules only accept non-missile labels, so missile precision doesn't change.
14. **A missile box on a truck, a radar or empty ground is still wrong.** That's 62 boxes: 39 on large trucks, 8 on radars, 9 on empty ground, 6 others.

### Rejected or not applied

- **E2:** P_Camo missile → vehicle (when loaded) is **flagged for visual check**, not accepted.
- **E5:** Transporter/Transloader → vehicle is **rejected**, because the model's behaviour is only partly consistent. Empty transporters are covered by R3 anyway.
- **E10:** radar ↔ missile is **rejected**; they are distinct classes.
- **MLRS → vehicle is not a separate rule.** It's accepted only through R3 (Is_Empty Yes/Nil).
- **P2f, P3 second labels and P4 relabel** are deferred to a later decision.
- **Any context rule for missile boxes on trucks in missile sites** is not applied; it's pending a visual check.

**Open items:**

- Visual check of the E2 cases (P_Camo, loaded, called vehicle: 8).
- Visual check of the MLRS under Is_Empty = Nil (31 vehicle matches).
- Visual check of the 40 missile boxes on large trucks.