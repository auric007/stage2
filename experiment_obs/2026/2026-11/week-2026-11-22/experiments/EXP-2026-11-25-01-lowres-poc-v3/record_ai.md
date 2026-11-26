# EXP-2026-11-25-01-lowres-poc-v3 — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-25_ai.md](../../daily/2026-11-25_ai.md).
Baselines: [EXP-2026-11-23-01](../EXP-2026-11-23-01-lowres-poc-train/record_ai.md) (v1), [EXP-2026-11-24-01](../EXP-2026-11-24-01-lowres-poc-v2/record_ai.md) (v2).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-25-01-lowres-poc-v3`

Status: **paused** — y11l-v3 done and scored (stopped on request at epoch 38, best = epoch 26). y8m-v3 stopped on request at epoch 8/45 (20:08 IST 25 Nov, val mAP50 0.337; `runs/y8m/weights/last.pt` resumable); y8m-v3 scoring and v3 ensemble not run. GPU given to EXP-2026-11-25-02.

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
- 13:08 IST: run 2 started from scratch (`runs/` cleared). 13:18 epoch 1 validated cleanly (val mAP50 0.179, 8 classes).
- y11l-v3 val mAP50 (8 cl.): ep9 0.417, plateau 0.38–0.417 to ep26 (0.417, best.pt by fitness), then ep27–38 0.38–0.41 (close_mosaic from ep36, no gain).
- 19:03 IST: researcher asked to stop training and evaluate the best epoch → y11l-v3 killed at epoch 38/45 (pipeline log shows "y11l FAILED" = the stop; annotated).
  The runner scored best.pt (epoch 26) on the 136 test scenes (19:03–19:11: `preds/y11l`, `reports/v3_y11l__merge_tank_spa.*`, `analysis/deep_v3_y11l.json`,
  `reports/compare_v123.md`), then started y8m-v3 (19:11 IST).

## Results and failures
y11l-v3 (epoch 26), 136 test scenes, 8-class space (v1/v2 tank+spa merged at eval), IoU 0.3, per-class best-F1 conf — `reports/compare_v123.md`:

| class | GT | v1 ens | v2 y11l | v2 ens | **v3 y11l** |
|---|--:|--:|--:|--:|--:|
| arty | 65 | 7.7 | 29.6 | 15.7 | 18.3 (P12/R35) |
| camo | 761 | 65.1 | 68.1 | 67.0 | 68.1 (P72/R65) |
| logistic | 690 | 34.1 | 34.2 | 34.5 | 33.7 (P36/R32) |
| missile | 307 | 45.6 | 46.8 | 48.4 | **48.9** (P51/R47) |
| radar | 239 | 52.2 | 57.3 | 58.9 | 57.7 (P73/R48) |
| smallvehicle | 1436 | 63.8 | 64.0 | 64.6 | **65.3** (P67/R64) |
| vehicle | 4136 | 69.0 | 69.3 | 69.7 | **70.5** (P73/R68) |
| tank_spa | 380 | 38.1 | 25.0 | 29.2 | 36.0 (P34/R38) |
| MICRO F1 | | 59.7 | 60.4 | 61.0 | **62.5** (P64.6/R60.6) |
| MACRO F1 (8) | | 47.0 | 49.3 | 48.5 | **49.8** |

IoU 0.5: MICRO 60.8 (P62.6/R59.1), MACRO 48.2.
- Single y11l-v3 beats every earlier single model and ensemble on MICRO (+1.5 vs v2 ens) and MACRO (+0.5 vs v2 y11l).
- tank_spa: 25.0 (v2 y11l) → 36.0; still below v1 ens 38.1 (success criterion not met yet; ensemble pending). Background tank_spa FPs at op conf
  194 (v2 ens) → 66: the background penalty worked; FPs now mostly on other objects: on vehicle 60 → 107, on camo 58 → 89 (277 FPs total vs 321).
  tank_spa GT (v2 ens → v3 y11l): found 120 → 144, → camo 48 → 33, → vehicle 65 → 68, low-conf 62 → 32, no box 29 → 39.
- radar 57.7 / missile 48.9 ≈ v2 (criterion "not below v2" met for missile, radar −1.2 vs v2 ens / +0.4 vs v2 y11l). Radar → vehicle 38 unchanged;
  missile → vehicle 56 unchanged, missile FPs on vehicle GT 91.
- arty 18.3 (v2 y11l 29.6; 65 GT): 23 found, 18 low-conf, 13 no box, 10 → vehicle; FPs 158 of 164 on background.
- logistic flat (33.7); logistic → vehicle 209 at op conf.

## Conclusions and next steps
Pending.

### Side analyses on y11l-v3 (2026-11-25 evening, researcher requests)
- Recall at a precision floor instead of best F1 (`code/recall_at_p.py`, conf grid step 0.005, IoU 0.3, `reports/recall_at_p.json`):
  radar R 47.3 (best F1, P 75) → 56.9 @P≥40 → 58.6 @P≥35; missile 47.9 (P 51) → 52.8 @P≥40 → 55.4 @P≥35. Any-precision ceiling (conf 0.05) radar 64.4, missile 64.8
  (v2 ens 68.6 / 73.9). At P≥35 the best is v3 y11l for radar (58.6) and v2 ens for missile (58.3).
- Multi-label radar/missile inference (`code/infer_multilabel.py`, `preds/y11l_ml`, `reports/recall_at_p_multilabel_y11l.json`): ceiling radar 64.4 → 66.1,
  missile 64.8 → 68.7, but R@P≥35 radar 58.6 → 57.3, missile 55.4 → 55.4 — added boxes score < 0.3; the → vehicle errors are confident, not near-misses.
- Child-wise confusion matrix (2026-11-26, researcher request, layout of `/sfs/e12_up42_lowres_child_confusion_with_background.png`): `code/child_confusion.py`
  → `analysis/v3_y11l_child_confusion.png` + `.tsv` (table). y11l-v3 ep26 at per-class F1-opt conf, original labels_simple parents / detailed child, IoU 0.3,
  columns = predicted class + 'below threshold' + 'detector missed'; background row = 1,377 detections ≥ conf on no labelled object (vehicle 393,
  smallvehicle 359, logistic 271, arty 158). Parent correct: vehicle 67.9 · smallvehicle 63.6 · camo 65.0 · logistic 31.9 (→ vehicle 209) · missile 46.6 ·
  tank 47.1 · radar 47.7 · arty 35.4 · spa 14.8 · aerial 0 (97 / 107 no box; not trained). Row n's equal E12's (same GT).
