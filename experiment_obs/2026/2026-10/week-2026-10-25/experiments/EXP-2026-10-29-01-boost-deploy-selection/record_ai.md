# EXP-2026-10-29-01-boost-deploy-selection — AI-authored

Author: AI agent (Claude). Generated: 2026-10-29T21:34:40+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-29-01-boost-deploy-selection`

Status: running. Phase 0 (CPU) is done; the Phase 2 GPU queue is running.

## Request summary and authorization

- **2026-10-29 ~21:30 IST.** The researcher approved [boost_plan_ai.md](../../boost_plan_ai.md):
  - the precision floors are confirmed;
  - **Option A**: keep Train359, no change to the /sfs/data split;
  - "start all experiments".
- **Goal:** boost the researcher-approved deployment selection in [knowledge/best_combination_ai.md §2](../../../../../knowledge/best_combination_ai.md).

## Question and design

- **Objective, confirmed:** maximise recall at a per-class precision floor, measured after the cross-class merge.
  - P ≥ 50: radar, missile, camo.
  - P ≥ 45: arty.
  - P ≥ 60: smallvehicle, tank, vehicle.
  - logistic: best F1.
- **Phase 0, CPU:**
  - E0a: an honest estimate of selection optimism, using site-disjoint cross-fitting.
  - E0b: merge rules.
  - E0c: fusion of saved prediction sets.
- **Phase 2, GPU** (`code/run_gpu.sh`):
  - A: a 10-epoch low-learning-rate fine-tune stage on the seed-0 new models, starting from last.pt and from best.pt.
  - B/C: seeds 1 and 2 of the EXP-2026-10-28-02 recipe for 100 epochs with **no early stopping**, a checkpoint every 10 epochs, the same fine-tune stage, and inference. Seed 1 also gets an epoch curve.
  - D: multi-seed ensembles of 6 and 3 models, using fuse `avgall` (sum / number of passes), because noisy-or saturates above 2 models.
- The relabels R, C and D are skipped (see the plan, point 5).

## Actual setup and changes

- **Data:** EXP-2026-10-28-02 `data/final` (Train359 + GE booster; val = Test116 tiles, as for m0001).
- **Scoring:** Test116, labels_simple, IoU 0.3, delete regions dropped. Env `/sfs/envs/yolo_models`; one L40S.
- **New code** in `artifacts/EXP-2026-10-29-01-boost-deploy-selection/code/`:
  - `train_seed.py`: the recipe with a seed argument, patience 0 and save_period 10;
  - `infer_set.sh` and `infer_avg.py`: a wrapper that adds the `avgall` fusion; the shared engine is unchanged;
  - `run_gpu.sh`: the GPU queue;
  - `sites.py`: 116 scenes → 90 sites, where footprints that overlap or lie within 3 km form one site → `state/test_sites.tsv`;
  - `scene_cache.py`: matched scores per scene; it reproduces the score_set sweeps (asserted);
  - `crossfit.py`, `merge_rules.py`, `fuse_sets.py`.
- **Fine-tune:** `engine_v1/train_finetune.py`, 10 epochs, lr0 2e-4, lrf 0.1, the m0001 augmentation.

## Execution

- 21:36 IST: GPU queue started (`nohup bash code/run_gpu.sh`); log in `logs/pipeline.log`. The fine-tune takes about 4 min per epoch.
- Estimated end ≈ 2026-10-31 late evening IST:
  - A: ~4.7 h;
  - each seed: 17.4 h of training + 1.75 h of fine-tune + ~1.5 h of inference;
  - D: ~2.5 h.
- Phase 0 ran on CPU while the GPU worked.

## Results — Phase 0 (CPU)

### E0a: how optimistic is choosing on Test116?

Method: 100 random site-disjoint halves × 2 directions. For each half, choose per class on the other half, score on this half ("honest"), and compare with choosing on the same half ("oracle", which is what our tables do).

Report: `reports/CROSSFIT_<pool>.md`.

| pool | MACRO F1 oracle → honest | MICRO F1 oracle → honest |
|---|---|---|
| all 25 sets, per-class mix | 58.8 → **53.1** (−5.7) | 67.2 → 63.7 |
| the 9 sets of the researcher's selection | 58.6 → **53.6** (−5.0) | 67.0 → 64.1 |
| new ensemble + new ensemble (last.pt) | 57.4 → 54.3 | 66.0 → 64.4 |
| new ensemble alone | 56.7 → **54.6** (−2.1) | 65.2 → 64.2 |
| m0001 ensemble alone | 55.7 → 54.2 | 65.1 → 64.4 |

- **Choosing among many sets per class overfits Test116.** Held out, the 25-set mix does worse than the new ensemble alone.
  - arty recall at P ≥ 45 drops 48.1 → 39.2.
  - tank F1 drops −9.
- **With floor-based picking, the floor is met on unseen sites only about 50% of the time.** The confidence sits exactly at the floor, so on new data it is a coin flip. Deployment needs a safety margin.

### Honest ranking of single sets

Only the confidence is chosen, on the other half. Full table: `reports/crossfit_single/`.

| set | weights | honest MICRO / MACRO | radar R@50 | missile R@50 | arty R@45 | tank R@60 |
|---|--:|---|--:|--:|--:|--:|
| **fz_newens_newlast** (new ens best.pt ⊕ new ens last.pt, box-mean fusion) | 4 | 65.0 / **55.6** | 62.4 | 56.4 | **50.2** | 79.5 |
| fz_newlast_m0001 | 4 | **65.4** / 55.4 | **63.1** | 58.3 | 44.8 | 78.8 |
| fz_newens_m0001 | 4 | 64.8 / 55.3 | 62.7 | 57.1 | 44.5 | 81.7 |
| new_ens | 2 | 64.2 / 54.6 | 61.5 | 56.4 | 45.9 | **81.9** |
| new_ens_last | 2 | 64.5 / 54.6 | 63.2 | 56.5 | 49.5 | 76.2 |
| old_m0001_ens | 2 | 64.4 / 54.2 | 60.6 | **59.2** | 41.1 | 80.1 |

- **More models fused beats picking per class.** Every 4-model fusion is above every 2-model set.
- This supports the GPU plan (seed ensembles) over finer per-class selection.

### E0b: merge rules

Report: `reports/MERGE_default.md`. Tested on the DEFAULT preset.

- **No rule beats the current one (M0) under the researcher's priorities.**
- Margin-only ranking (M2) raises MICRO 65.0 → 66.4, tank R +3.2 and vehicle R +4.1. But radar R falls 63.6 → 57.5 and missile R 56.2 → 51.1.
- The recall that tank and vehicle lose in the merge goes to arty, radar and missile boxes on the same objects. This is real class confusion (tank vs arty), not a merge artefact.
- **M0 is kept.**

## Conclusions so far (Phase 0)

1. The deployment selection's 65.0 / 57.3 is **optimistic by roughly 4–5 MACRO**. The honest estimate of that selection procedure is MACRO ≈ 53.6 (without the merge).
2. **Fusing more models is the lever.** Averaging the boxes of new ens best.pt and new ens last.pt (4 weight files) is honestly better (MACRO 55.6) than the 8-file per-class selection (≈ 53.6).
   - This is a candidate light/deploy option. It is the researcher's decision.
3. Phase 2 (seed ensembles, fine-tune stage) is the right direction. **Every Phase 2 result will be reported both honest (cross-fit) and in-sample.**
