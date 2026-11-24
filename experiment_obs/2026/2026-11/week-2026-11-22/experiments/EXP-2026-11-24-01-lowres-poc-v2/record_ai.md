# EXP-2026-11-24-01-lowres-poc-v2 — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-24_ai.md](../../daily/2026-11-24_ai.md).
Baseline: [EXP-2026-11-23-01](../EXP-2026-11-23-01-lowres-poc-train/record_ai.md). Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-24-01-lowres-poc-v2`

Status: **running** (queued 2026-11-24 ~19:10 IST; y11l-v2 → y8m-v2 → TTA test inference + scoring; ETA ~08:00–10:00 IST 25 Nov).

## Request summary and authorization
- Researcher, 2026-11-24 ~18:50 IST: deep-analyse the 136-scene test results (which classes fail and why; **radar and missile priority**;
  e.g. adjust the loss for classes that are confused with each other), consider the full range of fixes, then change model / parameters /
  training technique and **start the training again** (authorized). Test set, protocol and scoring unchanged.

## Analysis of v1 on the 136 test scenes (ensemble y8m+y11l, IoU 0.3, per-class best-F1 conf)
Script `EXP-2026-11-23-01/code/deep_test.py` → `analysis/deep_ens_y8m_y11l.json` (also `deep_y11l.json`). Per GT: TP / CONF>X (another class fires
at its operating conf) / LOWCONF (right class, below conf) / LOC / MISS (nothing ≥ 0.05). Per FP: on another class's GT / untrained label / dup / background.

**1. It is a classification problem, not a detection problem.** Share of GT with any box (any class, conf ≥ 0.05, IoU ≥ 0.3) vs with a same-class box:

| class | GT | F1 v1 ens | any-class ceiling | same-class ceiling |
|---|--:|--:|--:|--:|
| radar | 239 | 52.2 | 89.1 | 58.2 |
| missile | 307 | 45.6 | 94.5 | 69.4 |
| logistic | 690 | 34.1 | 94.9 | 51.4 |
| tank | 272 | 42.1 | 92.6 | 52.6 |
| spa | 108 | 26.0 | 84.3 | 45.4 |
| arty | 65 | 7.7 | 60.0 | 49.2 |
| camo | 761 | 65.1 | 95.5 | 81.7 |
| smallvehicle | 1436 | 63.8 | 94.6 | 82.6 |
| vehicle | 4136 | 69.0 | 95.8 | 88.3 |

**2. Confusion (GT row → predicted class at operating conf; missed = everything else):**
- radar 239: radar 102 · **vehicle 41** · missile 11 · camo 3 · missed 82 (LOWCONF 37, MISS 17, LOC 9, LOWCONF>vehicle 12).
- missile 307: missile 148 · **vehicle 36** · arty 8 · radar 5 · missed 109 (**LOWCONF 65**, LOWCONF>vehicle 15, MISS 16).
- the reverse direction costs precision: missile FPs = 129 on vehicle GT, 24 on radar GT, only 18 background; vehicle FPs = 296 on logistic, 145 on smallvehicle, 69 on missile, 49 on radar.
- other pairs: logistic→vehicle 198; tank→vehicle 51, tank→camo 38, camo→tank 42 (tank FPs 75 on camo GT); arty FPs 511 background, 91 % in 2 scenes.

**3. Radar detail (ens):** EWS 36 GT, recall 2.8 % (22 → vehicle; train has 118 EWS); FalcoPlus 5 (unseen in train) and Type305A 4 all → vehicle;
small radars 6–10 m recall 18 % (19/33 → vehicle) vs ≥ 15 m 55 %; SOC Pixels 46.9 % vs Site / Third_Party 30 %; GSD ≥ 0.72 m 11 % vs < 0.65 m 51 %;
PHR-1A/1B 6 % / 18 % vs WV02 49 % (mostly LOWCONF); two scenes (23JUN21043126, 23JUN21043007) hold 48 radar GT, 24 of them → vehicle.
Crops `analysis/radar_to_vehicle.jpg`: these are truck-mounted systems that look like trucks at 0.6 m.

**4. Missile detail (ens):** MLRS 33 GT recall 21 % (11 → vehicle); Transporter 13 recall 15 % (8 → vehicle); MissileSystem → arty 8 (one scene,
25JUN11035314); SPADA 7 → radar 4; Quality 02 (uncertain) recall 35 % vs 01 66 %; SOC Pixels 59 % vs Nil / Site / Third_Party 18–28 %;
GSD 0.65–0.72 m 19 % (mostly LOWCONF). Crops `analysis/missile_to_vehicle.jpg`, `analysis/missile_fp_on_vehicle.jpg`: several "vehicle" GT
hit by missile FPs look like launcher trucks (label ambiguity), many missile→vehicle misses are rows of trucks.

**5. Training-side cause:** in v1, radar was the least-seen class per epoch (3,561 instances vs missile 7,729, vehicle 70,559): RFS t = 0.15 left it ~1×.
v1 best.pt was picked by mAP50-95 only (ultralytics 8.4 default), which ignores the IoU-0.3 protocol and the radar/missile priority.

## Range of fixes considered
| lever | targets | status |
|---|---|---|
| confusion-aware weighted BCE (penalise the measured confusing class on each GT's anchors; up-weight radar/missile positives) | radar/missile↔vehicle, LOWCONF | **in v2** |
| higher cls loss gain 0.5 → 0.75 | classification bottleneck | **in v2** |
| radar ×3 / missile ×2.5 tile repeats | radar under-sampling | **in v2** |
| best.pt fitness with mAP50 + radar/missile AP50 | checkpoint choice | **in v2** |
| shorter schedule (45 ep, patience 15) | v1 peaked at epoch 19/23 | **in v2** |
| fine-type-aware sampling (EWS, MLRS, Transporter) | hardest sub-types | proposed (needs fine labels per tile) |
| soft / down-weighted targets for Quality 02 and SOC non-pixel radar/missile | label noise (recall 18–35 % there) | proposed (needs per-object weights in the loader) |
| crop classifier (stage 2b) on truck-like boxes with wide context | radar/missile/logistic vs vehicle | proposed — 89–96 % of GT already have a box |
| inference-side class re-ranking (e.g. pick radar/missile when its score ≥ k × vehicle score on the same box) | confusion at no training cost | proposed |
| per-sensor thresholds (PHR) | PHR recall 6–37 % | proposed |
| bigger backbone (YOLO11x-OBB) | fine-grained classes | not now (time budget) |
| label review: vehicle-labelled launcher trucks, EWS / Transporter policy | label ambiguity | researcher decision |
| arty background FPs (2 scenes) | arty precision | inspect scenes |

## Actual setup and changes (v2)
- Same tiles, preprocessing, DOTA-pretrained init, augmentation, batch, lr, imgsz 1024 and val (all 4,449 non-empty test tiles) as v1.
  Tiles reused by symlink from EXP-2026-11-23-01 (`data/tiles`, `data/test_tiles`).
- `code/cal_loss.py`: `ConfusionAwareOBBLoss` (subclass of ultralytics 8.4.41 `v8OBBLoss`): BCE element of a foreground anchor × M[assigned class, c].
  Positive weight radar 2.0, missile 2.0. Pair weights: radar→vehicle 2, radar→missile 2, radar→camo 1.5, missile→vehicle 2, missile→radar 2,
  missile→arty 2, missile→spa 1.5, vehicle→missile 2, vehicle→radar 2, vehicle→logistic 1.5, logistic→vehicle 1.5, tank→camo 1.5, camo→tank 1.5,
  tank→vehicle 1.5. Background anchors weight 1. Fitness = 0.5 mAP50 + 0.2 mAP50-95 + 0.3 mean AP50(radar, missile).
  Verified: criterion class is ConfusionAwareOBBLoss; box loss identical to the stock loss; cls loss on a radar/missile batch 1.65× stock.
- `code/make_v2_set.py`: v1 train tiles + v1 RFS realisation + floors radar ≥ 3, missile ≥ 2.5 (seed 2) → 20,097 tiles/epoch (v1 16,624).
  Instances/epoch radar 3,561 → 8,085, missile 7,729 → 14,682 (vehicle 70,559 → 90,873, as radar/missile tiles also hold vehicles).
- `code/train_v2.py`: cls 0.75, epochs 45, patience 15; else = v1 `train_one.py`. Smoke test: 1 epoch, 1 % data, 40 val tiles — ran clean.
- `code/run_v2.sh`: y11l-v2 (batch 12) → y8m-v2 (batch 16) → `infer_test.py` (same TTA) → `score.py` → `deep_test.py` → `compare.py` (`reports/compare_v1_v2.md`).

## Execution
- 2026-11-24 ~19:10 IST: `setsid nohup bash code/run_v2.sh` (waits for the CartoSat sweep of EXP-2026-11-23-01 to free the GPU).

## Results and failures
Pending.

## Conclusions and next steps
Pending results. Success criterion: radar and missile F1 above v1 (ens 52.2 / 45.6; y11l 49.8 / 44.2) without MICRO dropping.
