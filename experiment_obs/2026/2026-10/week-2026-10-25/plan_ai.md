# Week 2026-10-25 — plan (AI-authored)

Author: AI agent (Claude). Human companion: [plan.md](plan.md).

## PLAN (not approved, not started): smallest training subset; move the rest to test — TASK-20261028-train-subset

**Status:** approved 2026-10-28 ~01:00 IST, with these answers: y8m 15 ep; minimal drop; as many rare as possible; whole images; rare = radar, missile, arty; keep looping until stopped. Running as [EXP-2026-10-28-01](experiments/EXP-2026-10-28-01-train-subset-influence-loop/record_ai.md).

### 1. What the researcher wants (2026-10-28 ~00:40 IST)

- Dataset: `/sfs/data/up42_platform_data_27_10/prepared` — Train 359 images, Test1 44 + Test2 72 = Test116.
- **Test116 stays in test, unchanged.**
- **Find the smallest subset S of the 359 Train images that still gives the best test result.** Every removed image R = Train − S moves to test, making the test set bigger and the model better understood.
- **The moved images should be balanced across classes:** not only vehicle/smallvehicle, but also radar, missile, arty, logistic, camo and tank.
  - "Rear glass" in the request is read as one of these classes; to confirm.
- **Use existing influence techniques** to estimate how much each training image or instance helps the test set.
- **Run a sequential loop:** a "brain" decides the next run after each result.
- **Deliver the best small subset by the morning.**

### 2. Facts checked tonight

- **Contamination:** no Test116 image was in the old 27_08 Train316, so the current models (m0001 and the fine-tunes) are clean on Test116.
  - Test116 is: 43 old Test1, 34 old Test2, 1 ex-no_soc, 6 from 15_10 test, 32 new.
  - Train359 is: 316 old Train, 2 ex-Test2 (`1L`, `1Q`), 5 ex-no_soc, 20 from 15_10, 16 new.
- **Labels:** the 2026-10-27 rule. `labels_simple` has no P_Camo merge; radar/missile come from Parent Class; other classes come from the detailed class.
- **Train instances:**

| vehicle | smallvehicle | camo | logistic | tank | arty (+spa 447) | missile | radar |
|--:|--:|--:|--:|--:|--:|--:|--:|
| 12,287 | 5,349 | 2,742 | 1,915 | 1,802 | 730 | 711 | 421 |

- **MLRS:** 5 Train scenes hold all 100 Train MLRS. The /sfs/data rule says the researcher places MLRS scenes personally, so they stay in Train unless approved.
- **Known Train ↔ test footprint overlaps** (accepted earlier): 5 new + 12 old (data README). These are natural candidates to move, because they already sit next to test sites.
- **GPU:** 1× L40S 46 GB, idle. A y8m 10-epoch fine-tune on about 17k tiles took about 65 min on the V100; the L40S speed is not measured yet.
- **Generic starting weights (no UP42 data seen):** `/sfs/stage2/models/pretrained/{yolov8m-obb.pt, yolo11l-obb.pt}` (DOTA).

### 3. Ground rules of the study (proposed)

1. **Fixed yardstick:** sufficiency is judged only on Test116 (6,910 objects), which never changes. The moved images cannot be judged by the full-Train model, because it was trained on them.
2. **No leakage into the new test images:** images move as **site clusters**, i.e. footprint overlap or same pass, computed from the TIFF bounds. No moved image may share ground with a remaining Train image.
3. **Fair training start:** every subset model starts from the generic DOTA weights. Starting from m0001 would be unfair, because m0001 already saw every old Train image, so removing an image would not remove what was learned from it.
4. **Same recipe for every run:** only the image list changes. The GE booster is left out of the proxy: it is faster, and it makes the test stricter because the booster could hide what a removal costs.
5. **Metric:**
   - new `labels_simple`, one class per object, IoU 0.3;
   - per-class best-F1 on the 0.05–0.90 grid → MICRO/MACRO F1, plus recall at precision ≥ 0.30;
   - this is the `combine_best.py` scoring, pointed at the new dataset.
6. **"Still the best" = non-inferior** to the full-Train359 model trained with the same recipe:
   - MACRO F1 drop ≤ max(1.0, 2σ), where σ = seed-to-seed spread measured on the baseline;
   - no single class drops more than 3 F1;
   - (margins proposed; see section 7).
7. **No data is moved in /sfs/data by this study.** It produces the proposed move list; the researcher approves it before `prepared/build.py` placements change.

### 4. How influence is measured (no extra training needed after the baseline)

From the full-Train baseline, with checkpoints saved every few epochs:

| technique | what it gives | cost |
|---|---|---|
| **TracIn-CP** on the detection head: for each Train tile, the dot product of its loss gradient with the summed Test116 loss gradient, at 3–4 checkpoints, also split per test class | which images push the Test116 loss down (helpful) or up (harmful), per class | one forward + backward per Train tile per checkpoint, ~20–30 min |
| **Test-coverage (k-NN on embeddings):** backbone features of every GT instance; for each test instance, its nearest Train instances | images whose instances are never close to any test instance are redundant | ~10 min |
| **Training dynamics:** per-image loss over epochs (EL2N / forgetting) | easy / redundant vs hard / informative images | free from the logged checkpoints |
| **Class rarity** | removing the only examples of a rare type is risky | free |

These combine into a **removability score per site cluster**: low influence + redundant + common classes = remove first.

**Leave-cluster-out checks** (actual retrains) confirm the ranking at a few points. The scores are only a guide; the decision is always made by an actual retrain.

### 5. The loop ("brain" after every run)

- **Step 0, setup (CPU, ~1 h):**
  - 0.30 m state for 27_10 (manifest, test scene list, tiles for Train359 + Test116);
  - site clusters;
  - scorer for the new labels;
  - 1-epoch timing probe.
- **Step 1, baseline:** full Train359 proxy (+ a 2nd seed if time allows, for σ).
- **Step 2:** influence scores → ranked clusters.
- **Step 3 onward, one retrain per step:**
  1. Choose a removal level X (% of Train images): start at X = 30 %.
  2. Pick clusters by lowest removability, subject to a **class quota**: the moved set must hold at least q % of each class's Train instances (radar, missile, arty, logistic, camo, tank, plus vehicle/smallvehicle).
  3. Train on Train − R and score on Test116.
  4. Brain decision:
     - **pass** → raise X (bisection towards the largest passing X);
     - **fail** → find which class failed. If it is a quota class, lower that class's quota first; otherwise lower X.
     - At the first passing level, run **one random-removal control** of the same size to show the ranking beats chance.
  5. Every decision and result goes into the experiment record before the next run starts.
- **Run budget:** proxy y8m at 1024 px tiles, about 15 epochs, no booster. The estimate is ~45–70 min per run on the L40S (to be confirmed by the timing probe). From ~02:00 to ~09:00 that allows **baseline + 4–5 subset runs**. By morning this gives a **coarse answer**: the largest passing X among ~3 levels, plus one random control. It does not find the minimum to the single image.
  - 2 jobs can share the GPU (46 GB) to gain throughput.
  - A y8s proxy would allow ~8–10 runs but is a weaker stand-in for the product models.

### 6. What is delivered in the morning

- The best passing subset S and the move list R. R is given per scene, with class counts and site cluster; MLRS scenes are flagged.
- The proposed new test set = Test116 + R, with per-class totals and uniformity.
- Results table: full vs each subset vs the random control, MICRO/MACRO/per-class.
- The influence ranking of every Train scene (per class), reusable later.
- **Not by morning:**
  - retraining the full product (y8m + y11l + booster, all epochs, then the per-class combination) on S. This is the confirmation step after approval;
  - any change to /sfs/data.

### 7. Decisions needed from the researcher before starting

1. **Proxy / budget:** y8m 15 epochs (fewer, more faithful runs — recommended) or y8s (more runs)?
2. **Non-inferiority margin:** MACRO −1.0 (or 2σ) and per class −3 F1 — OK?
3. **Class quota q for the moved set:** e.g. ≥ 20 % of each rare class's Train instances; or just "as many as possible without failing"?
4. **Site clusters move together, and MLRS scenes stay in Train** — OK?
5. **"Rear glass"** — which class was meant?
6. **Caveat to accept:**
   - Choosing which images to remove by looking at Test116 makes the "still best on Test116" claim somewhat optimistic.
   - The moved images become clean extra test data, so the enlarged test set is still fair for future models.

### 8. Risks

- Seed noise can be as large as the effect: radar and missile have only 275 / 413 test objects. Two baseline seeds are needed to know σ.
- The proxy (single y8m, no booster) may not transfer exactly to the product combination. The final confirmation must use the product recipe.
- The setup step (new 0.30 m tiles for 27_10) is new engineering. If it overruns, the morning result shrinks to baseline + influence ranking + 1–2 subset checks.
