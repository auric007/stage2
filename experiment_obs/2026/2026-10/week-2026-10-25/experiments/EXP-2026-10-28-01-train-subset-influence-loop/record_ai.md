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

## Results

Pending; see [loop_status_ai.md](loop_status_ai.md).
