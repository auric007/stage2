# EXP-2026-11-25-01-lowres-poc-v3 — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-25_ai.md](../../daily/2026-11-25_ai.md).
Baselines: [EXP-2026-11-23-01](../EXP-2026-11-23-01-lowres-poc-train/record_ai.md) (v1), [EXP-2026-11-24-01](../EXP-2026-11-24-01-lowres-poc-v2/record_ai.md) (v2).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-25-01-lowres-poc-v3`

Status: **running** (run 2 started 2026-11-25 ~13:15 IST after a crash fix; y11l-v3 → score → y8m-v3 → ensemble → 8-class comparison; ETA ~00:00–02:00 IST 26 Nov).

## Request summary and authorization
- Researcher, 2026-11-25 ~13:00 IST: "start v3 and train with 8 classes by merging spa and tank and v3 change (spa weight and background penalty)". Authorized.
- Context: v2 raised radar / missile but spa collapsed (background FPs) and tank dropped; eval-time tank+spa merge of v2 ens gave tank_spa F1 29.2 (v1 ens 38.1).

## Question and design
- Does training with tank and spa as ONE class (tank_spa, 380 test GT), plus a positive weight and a background penalty on it and softer tank penalties,
  fix the v2 tank/spa regression while keeping v2's radar / missile gains?
- Comparison in the same 8-class space for all versions: `code/score_merge.py` (GT tank + spa → tank_spa; v1/v2 predictions remapped and
  de-duplicated IoU ≥ 0.4), `reports/compare_v123.md`. Reference (v2 ens, 8-class): MICRO 61.0, MACRO(8) 48.5, tank_spa 29.2, radar 58.9, missile 48.4.
  Success: tank_spa above v1 ens (38.1) and radar / missile not below v2.

## Actual setup and changes (vs v2)
- Data `code/make_v3_set.py`: same 20,097 train tiles/epoch (v2 list incl. radar/missile repeats) and 4,449 val tiles; images symlinked to v1 tiles;
  labels rewritten with 8 classes (spa index 8 → tank_spa 6). tank_spa instances/epoch 16,451 (tank 12,134 + spa 4,317).
- Loss `code/cal_loss.py`: positive weights radar 2.0, missile 2.0, **tank_spa 1.5**; **background weight tank_spa 2.0** (new: BCE of tank_spa on
  non-object anchors × 2); tank pairs softened: tank_spa→camo 1.25, camo→tank_spa 1.25, **tank_spa→vehicle removed** (v2: 1.5/1.5/1.5);
  missile→tank_spa 1.5; radar / missile / vehicle / logistic pairs as v2. Fitness = 0.5 mAP50 + 0.2 mAP50-95 + 0.2 mean AP50(radar, missile) + 0.1 AP50(tank_spa).
- Unchanged: DOTA-pretrained init, cls 0.75, epochs 45, patience 15, augmentation, imgsz 1024, batch 12 (y11l) / 16 (y8m), TTA inference.
- Checks: unit test (empty batch loss = 1.125 × stock, = background weight 2 on 1 of 8 classes; box/dfl/angle identical); 1-epoch smoke run with
  48 background + 6 object val tiles passes; 8-class scorer + deep analysis reproduce v2 merged numbers (tank_spa 29.2, MICRO 61.0, MACRO 48.5).
- Eval scripts: `code/infer_models.py` (8-class names), `code/score_merge.py tag=set`, `code/deep_test.py <tag> <set> <report>` (8-class space), `code/compare_v123.py`.

## Execution
- 2026-11-25 12:56 IST: run 1 `setsid nohup bash code/run_v3.sh`. 13:06 y11l-v3 **crashed** at epoch-1 validation (CUDA index-out-of-bounds in the TAL assigner): ultralytics resolves the val directory path, and `images/val` was a symlink to v1's folder → it read v1's 9-class val labels (spa = index 8). The runner then started y8m-v3; stopped. Smoke test had missed it (val given as a .txt list, which is not resolved). Logs `logs/failed_run1/`.
- Fix: `data/final/images/val` = real directory of per-file links; verified with ultralytics' own dataset loader that val (4,449) and train read v3 labels, classes 0–7 only.
- ~13:15 IST: run 2 started from scratch (`runs/` cleared).

## Results and failures
Pending.

## Conclusions and next steps
Pending.
