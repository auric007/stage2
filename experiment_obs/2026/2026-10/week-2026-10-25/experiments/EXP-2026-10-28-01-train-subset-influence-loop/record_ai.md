# EXP-2026-10-28-01-train-subset-influence-loop — AI-authored

Author: AI agent (Claude). Generated: 2026-10-28T00:52:23+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop`

Status: running (autonomous loop, until the researcher stops it).

## Request summary and authorization

TASK-20261028-train-subset. The plan is in [plan_ai.md](../../plan_ai.md).

**Researcher's approval and answers (2026-10-28 ~01:00 IST):**
- y8m, 15 epochs.
- The metric drop should be minimal.
- Move as many rare-class objects as possible without failing.
- Move whole image files, not patches.
- Rare classes = radar, missile, arty.
- Keep removing and trying until the researcher stops the loop; no fixed numbers, keep going in the optimal direction.

**Scope:**
- Training on the L40S and influence computation.
- No change to /sfs/data. The move list is a proposal.

## Setup (actual)

- **Dataset:** `/sfs/data/up42_platform_data_27_10/prepared` (Train 359, Test116 = 6,910 objects).
  - Labels: `labels_simple` under the 2026-10-27 rule, via `canon` (spa → arty).
  - Tiled at 0.30 m GSD, 1024 px, step 819: Train 12,522 tiles (4,158 with objects), Test 4,003 tiles. Took about 40 s (`code/prep_2710.py`).
- **Shared code change:** `/sfs/stage2/code/engine_v1/common.py` `DATASET` can now be overridden with env `STAGE2_DATASET`; the default is unchanged.
- **Site clusters** (`code/site_clusters.py`): lon/lat footprint bbox overlap → 300 clusters (257 singletons). The 5 MLRS scenes are pinned to Train. 18 clusters overlap test footprints.
- **Proxy recipe** (`code/train_subset.py`), the same for every run:
  - YOLOv8m-OBB from generic DOTA weights, imgsz 1024, 15 epochs;
  - AdamW lr0 1e-3, lrf 0.01, warmup 1, close_mosaic 3;
  - m0001 augmentations, no GE booster, no validation;
  - `last.pt` is the model; checkpoints every 5 epochs.
- **Scoring** (`code/infer_score.py`):
  - single-scale inference on Test116, conf ≥ 0.05, image-level class-wise NMS 0.4;
  - IoU 0.3, delete regions dropped;
  - per-class best F1 (conf 0.05–0.90) → MICRO/MACRO;
  - about 2.5 min per model.
- **Influence** (`code/tracin.py`, started by `code/tracin_waiter.sh`):
  - TracIn-CP on the detection head's final convs (17,259 parameters);
  - checkpoints epoch5 + epoch10, then last, of base_s0;
  - total test loss and per class.
- **Decision loop** (`code/brain.py`, background process, `touch .../STOP` to stop):
  - **pass rule:** MICRO/MACRO within max(0.5, seed noise); radar/missile/arty within max(2.0, noise); other classes within max(3.0, noise);
  - two proposals in parallel (A = next k images by lowest influence per image, B = first half of A);
  - A passes → accept and double k; B passes → accept B; both fail → mark the clusters carrying the failing classes' influence as KEEP and halve k;
  - one random-removal control;
  - then retry KEEP clusters one at a time, then confirm with seed 1.
  - Dry-run tested on a scratch copy with fake metrics before the real start.

## Execution log

- 00:55 IST: tiling done.
- ~00:58: `base_s0` and `base_s1` (full Train359, seeds 0/1) started together, about 7 min per epoch each.
- 01:00: reference scoring of m0001's y8m alone on Test116: MICRO 61.3 / MACRO 53.4 (radar 53.4, missile 55.5, arty 48.0).
  - m0001 was trained on old Train316 + booster, 100 epochs.
- 01:06: brain started; it waits for the baselines (~02:45) and the first influence file.
- **Live status, updated after every decision:** [loop_status_ai.md](loop_status_ai.md).
- ~02:20: TracIn on epoch5+epoch10 **crashed with CUDA out of memory**. It started while both baselines held 22 GB each.
  - Re-launched at 02:41, after training ended (`logs/tracin_e5_e10_oom.log` keeps the failure).
- 02:41: baselines scored (Test116, new `labels_simple`, IoU 0.3, per-class best F1):
  - seed 0: MICRO 58.3 / MACRO 46.0;
  - mean of the two seeds: **MICRO 57.9 / MACRO 45.8**;
  - seed noise: 0.7 / 0.5.
- 02:43: TracIn on `last` done. **Step 1** launched from this file alone:
  - A = 143 images (121 clusters); B = 71 images (60 clusters).
- 02:43: **proposal A crashed before training** with `EOFError` reading `data/tiles_norm/labels.cache`.
  - Cause: ultralytics writes a single label cache next to the shared labels dir, and every run with a different image list rewrites it, so concurrent runs race.
  - Fix: `train_subset.py` now gives each run its own `runs/<run>/ds/{images,labels}` symlink view, so each run has its own cache.
  - A was re-launched at 02:50 with the same keep list and seed; the original crash log is `logs/train_s01_A_rm143_cacherace.log`.
  - The brain had not yet judged step 1, so no decision used the crash.
- 03:00: **TracIn weighting bug fixed.**
  - `last.pt` stores `epoch = -1` after training, so tracin.py weighted `last` with lr0 = 1e-3 instead of the final 1e-5.
  - `tile_influence_last.json` was rescaled ×0.01 (noted in its `meta`), and `tracin.py` was fixed.
  - Step 1's ranking is unaffected: one file, uniform scale.
  - From step 2 on, the ranking is TracIn-CP over epoch5 + epoch10 + last, once the epoch-5/10 file is written.

- 03:58: **v1 step 1: both proposals FAIL** (fixed 15 epochs):

| run | images removed | tiles | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| base_s0 | 0 | 12,522 | 58.3 | 46.0 | 26.8 | 52.9 | 17.5 | 44.7 | 46.6 | 55.2 | 54.5 | 69.9 |
| base_s1 | 0 | 12,522 | 57.5 | 45.6 | 22.5 | 50.2 | 19.1 | 45.5 | 45.7 | 56.5 | 56.1 | 69.0 |
| v1 s01_A | 143 | 7,350 | 49.6 | 39.1 | 17.0 | 45.0 | 18.4 | 39.5 | 39.0 | 48.2 | 44.3 | 61.3 |
| v1 s01_B | 71 | 10,135 | 55.0 | 43.1 | 21.1 | 50.0 | 17.8 | 45.5 | 38.9 | 47.6 | 55.8 | 67.8 |

- **Diagnosis: a design confound, not proof that the images are needed.**
  - Training ran a fixed number of epochs, so removing 41% of the tiles also removed 41% of the optimizer steps.
  - The proxy is still improving at epoch 15: base_s0 box loss 1.26 → 1.20 and cls loss 1.85 → 1.73 over epochs 10 → 15.
  - A dropped roughly uniformly, 5–10 F1 in every class except logistic, which fits undertraining.
  - B's losses sit mostly in radar (−7.2) and smallvehicle (−8.3), so part of it may be real; v2 separates the two effects.
- ~04:00: **v1 stopped.** The brain and the v1 step 2 (A 71 / B 38, launched from the confounded step-1 KEEP marks) were killed.
  - v1 is archived in `runs_v1_epochmatched/` and `state/v1_epochmatched/` (brain state, loop log, keep lists).
  - The notebook snapshot is [loop_status_v1_epochmatched_ai.md](loop_status_v1_epochmatched_ai.md).
- 04:01: **loop v2, equal optimizer steps.**
  - Each subset trains 15 × (12,522 / its tiles) epochs; warmup, close_mosaic and save_period are scaled by the same factor.
  - The full set's factor is 1, so the baselines are unchanged and still valid.
  - Each run writes `runs/<run>/steps.json`.
  - The brain restarted from a fresh state; step 1 uses the 3-checkpoint TracIn ranking:
    - A: 143 images, 7,259 tiles, 26 epochs;
    - B: 71 images, 9,861 tiles, 19 epochs.
  - The researcher's "y8m for 15 epochs" is kept as the budget of the full set, i.e. 15 full-Train epochs of compute for every run.
  - Cost: each step now takes as long as a baseline, about 105 min for two parallel runs. That is about 3 steps by 09:30.

- 05:46: **v2 step 1 (equal steps): both FAIL, but MACRO held for the larger removal.**

| run | images removed | epochs | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| baseline (2-seed mean) | 0 | 15 | 57.9 | 45.8 | 24.7 | 51.6 | 18.3 | 45.1 | 46.2 | 55.9 | 55.3 | 69.5 |
| v2 s01_A | 143 | 26 | 54.6 | **45.6** | **31.2** | 52.9 | 18.0 | **47.1** | 43.1 | 50.1 | **58.5** | 64.1 |
| v2 s01_B | 71 | 19 | 56.3 | 43.3 | 17.9 | 47.9 | 18.8 | 44.9 | 41.3 | 49.7 | 56.7 | 69.0 |

  - Equal steps removed most of the v1 gap: A went from MICRO 49.6 / MACRO 39.1 to 54.6 / 45.6.
  - A fails on MICRO, radar (−3.1), smallvehicle (−5.8) and vehicle (−5.4).
  - **Selection was unbalanced.** Ranking by influence per image favours dense images: A moved 40% of the images but 50% of vehicle, 53% of smallvehicle and 42% of radar objects. B moved 20% of the images but 33% / 39% / 21%.
  - B (smaller) scored below A on MACRO: single-run noise on small classes is larger than the 2-seed estimate suggests (arty 17.9 vs 31.2).
- 05:49: **v2b, class-balanced proposals.**
  - The brain's automatic step 2 (A 71 / B 37, unbalanced, nearly a repeat of s01_B) was killed; its runs are in `runs_killed/`.
  - `brain.py` now uses `take_balanced`: it walks the same low-influence order and skips any cluster that would push a class's moved share above the moved image share + 3 pts.
  - k was reset to 143. The KEEP marks C011 and C200 from step 1 were kept (conservative); retries test them later.
  - Step 2 proposals:
    - A: 143 images moved; moved share per class arty 22, camo 34, logistic 37, missile 37, radar 42, smallvehicle 39, tank 11, vehicle 43 %;
    - B: 71 images moved; 10–23 % per class.

- 07:34: **v2b step 2 (balanced):**

| run | images removed | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle | verdict |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|---|
| baseline | 0 | 57.9 | 45.8 | 24.7 | 51.6 | 18.3 | 45.1 | 46.2 | 55.9 | 55.3 | 69.5 | — |
| s02_A | 143 | 54.5 | 43.2 | 21.4 | 49.6 | 20.0 | 42.8 | 41.2 | 48.3 | 56.6 | 65.9 | FAIL (MICRO, MACRO, missile, radar, smallvehicle, vehicle) |
| s02_B | 71 | 57.5 | 45.5 | **16.8** | 48.8 | 19.8 | 44.6 | 45.6 | 54.0 | 65.2 | 68.9 | FAIL (arty only) |

  - Moving 71 balanced images is within margin on everything except arty (−7.9).
  - Arty is the noisiest class: baseline seeds 26.8 / 22.5 (seed-noise estimate 4.3); single subset runs so far range 16.8–31.2.
  - Balanced 143 is worse than unbalanced 143 (v2 s01_A, MACRO 45.6) on MACRO, so a single run at this size cannot tell the two apart.
  - The brain marked the 2 clusters carrying most positive arty influence (C067, C125) KEEP, set k = 71 and launched step 3: A 71, B 35 images.

- 09:20: **v2b step 3: first accepted subset.**

| run | images removed | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle | verdict |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|---|
| baseline | 0 | 57.9 | 45.8 | 24.7 | 51.6 | 18.3 | 45.1 | 46.2 | 55.9 | 55.3 | 69.5 | — |
| s03_A | 71 | **58.0** | **46.4** | 26.4 | 50.0 | 18.3 | **50.0** | **47.2** | 52.9 | 57.2 | 69.2 | **PASS** |
| s03_B | 35 | 54.3 | 43.2 | 17.0 | 51.5 | 19.2 | 45.9 | 42.6 | 53.3 | 51.0 | 64.8 | FAIL |

  - **Accepted move set:** 71 images (63 clusters, 2,733 tiles), saved in `state/best_removed_scenes.txt`.
    - Objects moved, of Train totals: arty 119/1,177 (10%) · camo 452/2,742 (16%) · logistic 356/1,915 (19%) · missile 78/711 (11%) · radar 53/421 (13%) · smallvehicle 1,218/5,349 (23%) · tank 122/1,802 (7%) · vehicle 2,798/12,287 (23%).
  - **Single-run noise is about ±3 MICRO.**
    - s03_B removed fewer images (35, not a strict subset of A: 6 different clusters) yet scored 3.7 MICRO below A.
    - Both trained normally, with matched steps and final losses 1.18 / 1.61 vs 1.16 / 1.53.
    - The 2-seed baseline spread (0.7) underestimates the true run-to-run variance, so one PASS or FAIL is weak evidence.
  - The brain accepted A, doubled k to 142 and launched step 4:
    - the random control (71 random images);
    - A: 214 images.
- 09:23: **v2c, seed-1 confirmation of accepted levels.**
  - From step 5, the newest unconfirmed accepted level is re-trained with seed 1 in place of B.
  - It is kept only if the 2-seed mean passes the same rules against the 2-seed baseline. Otherwise R reverts to the previous level, that step's A/B results are ignored, and k is set to half the reverted chunk.
  - Tested on a scratch copy with fake metrics: accept, confirm-fail → revert → re-confirm of the earlier level, retries, final confirm.
  - Deployed without interrupting the running step-4 jobs. `state/brain_state.json` was seeded with the level `s03_A_rm71`.

- 11:06: **step 4: random control and 214 images.**

| run | images removed | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle | verdict |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|---|
| baseline | 0 | 57.9 | 45.8 | 24.7 | 51.6 | 18.3 | 45.1 | 46.2 | 55.9 | 55.3 | 69.5 | — |
| s03_A (influence, balanced) | 71 | 58.0 | 46.4 | 26.4 | 50.0 | 18.3 | 50.0 | 47.2 | 52.9 | 57.2 | 69.2 | PASS |
| s04_random (seed-0 random clusters) | 71 | 55.2 | 42.9 | 16.9 | 49.0 | 17.7 | 47.6 | 40.1 | 51.9 | 51.5 | 68.2 | FAIL |
| s04_A | 214 | 52.4 | 42.9 | 31.9 | 47.0 | 17.6 | 49.7 | 45.5 | 47.5 | **39.9** | 64.1 | FAIL |

  - **The influence ranking beats random at 71 images**, by +2.8 MICRO and +3.5 MACRO. This is one run each, and single-run noise is about ±3, so it is indicative, not conclusive.
  - At 214 images, tank collapses (−15.4) and vehicle, smallvehicle and camo fail; arty and missile hold.
  - The brain marked 7 clusters KEEP (C294, C189, C249, C181, C121, C193, C033) for the failing classes and set k = 71.
  - **Step 5** (11:06): seed-1 confirmation of the 71-image level (`s05_confirm_s03_A_rm71`) and A = 142 images.

- 12:50: **step 5: the 71-image level is CONFIRMED on 2 seeds.**

| | MICRO | MACRO | arty | camo | logistic | missile | radar | smallveh. | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| baseline, 2-seed mean (Train359) | 57.9 | 45.8 | 24.7 | 51.5 | 18.3 | 45.1 | 46.1 | 55.9 | 55.3 | 69.5 |
| 71 moved, seed 0 (s03_A) | 58.0 | 46.4 | 26.4 | 50.0 | 18.3 | 50.0 | 47.2 | 52.9 | 57.2 | 69.2 |
| 71 moved, seed 1 (s05_confirm) | 57.4 | 45.9 | 25.4 | 47.2 | 18.6 | 48.0 | 44.4 | 53.8 | 60.6 | 69.5 |
| **71 moved, 2-seed mean** | **57.7** | **46.2** | 25.9 | 48.6 | 18.4 | 49.0 | 45.8 | 53.3 | 58.9 | 69.4 |
| difference | −0.2 | +0.4 | +1.2 | −2.9 | +0.1 | +3.9 | −0.3 | −2.5 | +3.5 | −0.1 |

  - Camo is the only class near its margin (−2.9 against the −3.0 limit); the seed-1 run alone failed camo at 47.2.
  - s05_A (142 images): MICRO 55.2 / MACRO 45.3, FAIL on MICRO, camo, smallvehicle and vehicle; tank held at 55.3.
  - The brain marked 7 more clusters KEEP (18 in total) and set k = 35.
  - Step 6: A 106 images, B 88 images.

- 14:37: **step 6: both FAIL.**
  - s06_A (106 images): MICRO 57.1 / MACRO 45.3; arty 22.2, camo 49.0, missile 46.0, radar 45.3, smallvehicle **49.4**, tank 62.6, vehicle 69.4.
  - s06_B (88 images): MICRO 55.1 / MACRO 44.0; smallvehicle 52.2, tank 49.6, vehicle 65.7.
  - Again the smaller removal scored lower. The brain marked C018 KEEP (19 in total) and set k = 17.
- 14:39: **v2d, paired seeds.**
  - The single-seed step 7 (A 88 / B 79) was killed (`runs_killed/`).
  - Each proposal now trains with seeds 0 and 1 in parallel and is judged on the 2-seed mean against the 2-seed baseline. A passing level counts as confirmed.
  - The search stops once k < 8 images (`KMIN`): a few images cannot be resolved against about ±2 MICRO of noise in the 2-seed mean.
  - Tested on a scratch copy with fake metrics.
  - Step 7: 88 images (the 71 level + 15 clusters), seeds 0 and 1.

## Results

Pending; see [loop_status_ai.md](loop_status_ai.md).
