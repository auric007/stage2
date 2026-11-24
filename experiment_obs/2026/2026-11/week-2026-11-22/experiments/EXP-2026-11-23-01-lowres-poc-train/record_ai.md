# EXP-2026-11-23-01-lowres-poc-train — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-23_ai.md](../../daily/2026-11-23_ai.md).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-23-01-lowres-poc-train`

Status: **completed** 2026-11-24 07:41 IST (final y8m start 2026-11-23 20:18 IST, after three restarts; see Execution).

## Request summary and authorization
- Researcher, 2026-11-23 ~20:00 IST: "can you start training, I want to do it asap; find how we can complete the training in 12–16 hrs".
  Approved preprocessing: colour fix + 2–98 % stretch everywhere; HR degraded to test GSD; GSD augmentation on half of LR; Delete masked in
  training tiles; repeat-factor sampling. Moderate RFS (t = 0.15) taken as default. The 10 LR train scenes > 0.80 m were first kept by default, then **dropped by the researcher (~20:17 IST)**.
- Data: `/sfs/data/lowres_poc/up42_complete` after the 2026-11-23 label rules (D-20261123-01; [labels record](../../daily/2026-11-23_ai.md)).

## Question and design
- Train the best stage-2 recipe (YOLOv8m-OBB + YOLO11l-OBB, the members of every HYBRID group) on the new split (train 420 LR + 416 HR,
  test 136 LR at 0.58–0.80 m) and measure it on the 136 test scenes. 9 classes (spa added).
- Init from DOTA-pretrained weights. Not from HYBRID weights: they were trained on 02_11 Train359, which holds 91 of these 136 test scenes.
- No earlier model is a clean baseline on this test (same reason).

## Actual setup and changes
- Code (not versioned): `code/prep.py`, `code/train_one.py`, `code/run.sh`, `code/infer_test.py`, `code/score.py`, `code/infer_test.sh`.
  Env `/sfs/envs/yolo_models`; GPU 1× L40S 46 GB.
- **Image reader (new):** RGB bands from metadata `eo:bands` (the engine_v1 reader swaps R/B on PHR-1A/1B: 21 test, 7 train scenes) and per-channel
  2–98 % stretch over valid pixels, same for train, val and test inference. Visual check: `reports/prep_check.jpg`.
- **Resolution:** target 0.30 m. Test and undegraded train: native → bilinear. HR (416): INTER_AREA down to a GSD drawn from the 136 test GSDs, then
  bilinear to 0.30 m. LR: 208 of 420 degraded to U(0.60, 0.80) the same way (seeded per scene). The 10 LR scenes > 0.80 m (125 trained-class objects) are excluded.
- **Delete:** blacked out in train/val tiles; objects with centre in Delete dropped (train 60,441 → 59,957; test kept for scoring, 8,014).
- **Tiles:** 1024 / 819; object kept if ≥ 40 % inside. Train 19,275 tiles → 9,472 with objects + 5,100 background (35 %) = 14,572 unique.
  **RFS** t = 0.15: arty ×3.12, spa ×3.51, tank ×2.01, missile ×1.33, radar ×1.17, camo/logistic ×1.16 → 16,686 tiles per epoch (+2,114 repeats).
  Val (best.pt only): **ALL 4,449 non-empty test tiles** (researcher 20:10: "validate all test scenes", as last time: EXP-2026-10-28-02 validated on all Test116 tiles).
  Final train set: **16,624 tiles per epoch (+2,141 repeats)**, from 9,414 object tiles + 5,069 background; RFS arty ×3.11, spa ×3.50, tank ×2.00.
- **Recipe:** EXP-2026-10-28-02 hyper-parameters unchanged (AdamW lr0 0.001×b/16, lrf 0.01, imgsz 1024, mosaic 1, mixup 0.1, degrees 180, …), except
  **epochs 60** (a 70-epoch start measured ~5.5 min/epoch for y8m and was aborted after 2 min to fit the budget), patience 30, cache RAM, workers 16.
  y8m batch 16, y11l batch 12, seed 0, sequential.
- **Test inference:** 4,831 non-empty 0.30 m tiles of the 136 test scenes; TTA 1024/1536 × flip none/h; noisy-or per tile per class at IoU 0.4; scene dedup;
  sets y8m, y11l, ensemble. Scoring: test labels_simple (9 classes), Delete by centre, greedy rotated IoU 0.3 (and 0.5), per-class conf sweep.
  Pipeline check: GT fed as predictions scores 100 % (3 of 8,014 missed: centre in blank/Delete area).

## Execution
- 20:01 prep (33 s, 96 workers). 20:02 `nohup bash code/run.sh` (70 epochs) → 20:04 aborted and restarted with 60 epochs.
- 20:10 researcher: validate on all test scenes → 20:12 restart (val subset 2,022 → all 4,449). That restart was aborted at 20:13: the old process was still
  on the GPU, and the train set was a union of two random selections (prep collected scenes in completion order). Fixed: prep sorts scenes (selection is
  deterministic; two rebuilds gave the same file list md5), `data/final` rebuilt from scratch. 20:14 start of `code/run.sh`.
- ~20:17 researcher: drop the 10 LR train scenes > 0.80 m → y8m stopped (leftover dataloader workers force-killed), `data/final` rebuilt,
  **20:18 final start**.
- Expected (estimate from the first epoch, not measured end-to-end): y8m ~5.5 min/epoch → ~01:50 IST; y11l ~8.5 min/epoch → ~10:20; TTA inference + scoring
  ~1–1.5 h → results ~11:30 IST 2026-11-24 (~15 h total).
- Logs: `logs/pipeline.log`, `logs/train_y8m.log`, `logs/train_y11l.log`, `logs/infer.log`. Reports: `reports/{y8m,y11l,ens_y8m_y11l}.md`.

## Results and failures
**Timeline (observed, `logs/pipeline.log`):** y8m 20:18 → 00:39 (early stop at epoch 49, best epoch 19 by fitness; 4.3 h);
y11l 00:39 → 07:29 (early stop at epoch 53, best epoch 23; 6.8 h); TTA inference + scoring 07:29 → 07:41. Total 11.4 h. No failures.
Both models plateau early on val (= all test tiles): patience 30 stopped them ~30 epochs after the best.

**Test, 136 scenes, IoU 0.3, per-class best-F1 conf** (`reports/*.md`; grid extended to 0.925–0.99 on 2026-11-24 because the ensemble's best confs
sat at the old 0.9 ceiling):

| set | MICRO P / R / F1 | MACRO F1 | IoU 0.5 MICRO F1 / MACRO |
|---|---|--:|---|
| y8m | 56.2 / 58.7 / 57.4 | 41.6 | 57.0 / 39.7 |
| y11l | 61.9 / 59.0 / 60.4 | 44.2 | 58.0 / 41.6 |
| **ensemble y8m + y11l** | 59.8 / 59.6 / 59.7 | **45.1** | 58.0 / 42.8 |

Ensemble per class (P / R / F1): vehicle 70.4/67.6/69.0 · camo 69.4/61.4/65.1 · smallvehicle 67.1/60.9/63.8 · radar 67.1/42.7/52.2 ·
missile 43.3/48.2/45.6 · tank 47.5/37.9/42.1 · logistic 34.6/33.6/34.1 · spa 27.0/25.0/26.0 · arty 4.3/36.9/7.7.

**False-positive breakdown (ensemble at the best conf):**
- arty: 516 of 541 FPs are on background, 488 of them in two scenes (`25JUN16072810…` 347, `IMG_PHR1B_PMS_…` 141). Unannotated objects or a
  look-alike texture there: not inspected yet (unknown).
- tank: 75 FPs on GT camo (netted tanks are labelled camo by Parent Class 05), 22 on vehicle.
- missile: 129 FPs on GT vehicle, 24 on radar. logistic: 250 on background, 158 on vehicle. spa: split across background/camo/vehicle.
- Not comparable with any earlier stage-2 number: different test set (136 lowres_poc scenes, 9 classes).


### External test: CartoSat, 7 scenes (2026-11-24, `/sfs/data/lowres_poc/cartosat_test`, converted in another session)
- 0.6 m pansharpened 8-bit; band order from metadata `tif_band_order` (2 scenes B,G,R, 5 R,G,B; verified against the provider RGB PNG by band
  correlation, all 7 agree). Same pipeline (0.30 m, TTA); 13,028 tiles; 13:58 → 14:31 IST. Code `code/infer_external.py`, `code/score_external.py`.
- GT 665 trained-class objects (vehicle 360, smallvehicle 267, rare classes 2–9 each); 94 lie inside lowres_poc train scenes → second scope ignores them.
- Scoring fixes during this run: per-class conf ties (F1 = 0 or no GT) now take the highest conf (were 0.05, which added up to 1,038 FP to MICRO).
- IoU 0.3, MICRO P / R / F1 · MACRO (`reports/cartosat_*.md`):
  - y8m: all 9.3 / 29.6 / 14.2 · 9.7; excl. train sites 12.2 / 29.2 / 17.2 · 9.6
  - **y11l: all 21.9 / 40.0 / 28.3 · 14.3; excl. 29.7 / 36.4 / 32.7 · 10.3**
  - ensemble: all 14.5 / 34.7 / 20.5 · 14.0; excl. 23.8 / 34.3 / 28.1 · 10.5
  - y11l per class (all): vehicle 42.7/40.6/41.6, smallvehicle 25.9/41.6/31.9, camo 40.0/66.7/50.0 (9 GT); arty/tank/spa 0 found; missile/radar/logistic ≤ 1 found.
- Checks: labels are aligned (median GT→nearest-prediction offset < 1 px in every scene with hits). Recall by scene (any class, y11l ≥ 0.5): 45RYM 92/123,
  42SXB 140/257, 46RCT 67/140, 50QJL 25/93, 25493 5/30, 41906 1/17. Missed-object crops (`reports/cartosat_missed.jpg`): snow scenes (25493, 41906, strong
  colour noise in 41906) and dense car rows in 50QJL where only some cars are labelled (observed) → precision here is a lower bound (supported inference).

## Conclusions and next steps
- The recipe trains within budget (11.4 h incl. inference). y11l is the stronger single model; the 2-model ensemble gives the best MACRO (45.1).
- Weakest: arty (FPs concentrated in 2 test scenes), spa, logistic, tank (camo/tank ambiguity under nets).
- Proposed (not run): inspect the two arty-FP scenes; confusion analysis; per-class model/threshold selection; extra seeds.
