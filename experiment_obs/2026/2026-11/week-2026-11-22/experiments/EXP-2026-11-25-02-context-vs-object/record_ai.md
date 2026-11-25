# EXP-2026-11-25-02-context-vs-object — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-25_ai.md](../../daily/2026-11-25_ai.md).
Related: [EXP-2026-11-25-01](../EXP-2026-11-25-01-lowres-poc-v3/record_ai.md) (v3, 8 classes; y11l-v3 epoch 26 = the probed detector).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-25-02-context-vs-object`

Status: **running** (started 2026-11-25 20:11 IST; phase 0 + phase 1 in parallel on the L40S).

## Request summary and authorization
- Researcher, 2026-11-25 ~20:00 IST: wants to know whether the model classifies the 8 classes from the object's own pixels (fine features) or from
  context / placement; proposed graying out GT box pixels and training on the surroundings only, then larger patches for more context.
- AI proposed a plan (phase 0 = probe of the current detector with context removed; phase 1 = per-instance classifier ablation; optional 1b =
  literal grayed-GT detector; phase 2 = act on results). Researcher ~20:07 IST: **approved phase 0 + phase 1 as designed, run now; stop the
  y8m-v3 training** (done, EXP-2026-11-25-01 paused at y8m epoch 8). Phase 1b not answered → not run (decide after phase 1).

## Question and design
- Why not the literal grayed-GT detector: the gray box gives away location (oracle localisation, test must be grayed at GT) and shape
  (size / aspect / angle — aspect already separates logistic vs vehicle); shadows leak outside the box; 58 / 136 test scenes overlap a train
  footprint (`/sfs/data/lowres_poc/analysis/geo_overlap.tsv`) → a context model can memorise sites; ~5 h per variant.
- **Phase 0** (no training): y11l-v3 (epoch 26) on the same 4,831 test tiles with every pixel farther than r m from any labelled object set to
  gray 114, r = 2 / 10 / 30 / 100 m, same TTA + fusion. Per GT at the v3 op conf: same-class recall, detected (any class), class accuracy
  given detected; all / clean (78 scenes) / overlap (58). `code/phase0_rings.py`, `code/phase0_score.py`.
- **Phase 1**: one 640 px (192 m) crop per GT instance from the same 0.30 m renderings as the detectors (`code/make_crops.py`); 8-class
  ResNet-50 (ImageNet) per arm (`code/train_cls.py`): FULL (object + surroundings), CONTEXT (target OBB +1.5 m grayed, neighbours visible),
  SITE (all labelled objects grayed), OBJECT (only target +1.5 m), SHAPE (silhouette only = leak control); FULL and CONTEXT at 48 / 96 / 192 m,
  SITE 96, OBJECT 48, SHAPE 48 (9 runs). Balanced sampling, best epoch by val macro-F1, test once with 4-rotation TTA.
  Read-out per class (recall, confusion) on the 78 clean scenes, plus all / overlap, SOC and Quality breakdowns, scene-bootstrap CIs (`code/eval_cls.py`).

## Actual setup
- Crops: val = 83 of the eligible train scenes (seeded stratified search, every class 8.8–11.2 % in val); vehicle / smallvehicle / camo capped
  at 5,000 train / 600 val. Train 26,369 (vehicle, smallvehicle, camo 5,000 · logistic 3,507 · tank_spa 2,994 · missile 2,598 · radar 1,415 ·
  arty 855), val 3,069, test 8,014 (= all 8-class test GT, same set as score.py). Train scenes: Delete blacked out, LR > 0.80 m excluded (as v1–v3).
- Checks: mask renderings verified visually (`analysis/phase0_mask_check_radar_tile.jpg`, `analysis/phase1_arms_check_radar.jpg`);
  smoke run (train → val → test TTA → report) passes.

## Execution
- 20:08 IST y8m-v3 stopped (EXP-2026-11-25-01). 20:11 phase 0 started (`code/run_phase0.sh`). 20:11–20:15 crops built (CPU).
- 20:15 IST phase 1 started (`code/run_phase1.sh`; order FULL96, CONTEXT96, OBJECT48, SHAPE48, SITE96, FULL48, FULL192, CONTEXT48, CONTEXT192).

## Results and failures
Pending.

## Conclusions and next steps
Pending.
