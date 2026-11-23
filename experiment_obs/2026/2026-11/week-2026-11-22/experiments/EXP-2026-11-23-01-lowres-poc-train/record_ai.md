# EXP-2026-11-23-01-lowres-poc-train — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-23_ai.md](../../daily/2026-11-23_ai.md).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-23-01-lowres-poc-train`

Status: **running** (final y8m start 2026-11-23 20:14 IST, after two restarts; see Execution).

## Request summary and authorization
- Researcher, 2026-11-23 ~20:00 IST: "can you start training, I want to do it asap; find how we can complete the training in 12–16 hrs".
  Approved preprocessing: colour fix + 2–98 % stretch everywhere; HR degraded to test GSD; GSD augmentation on half of LR; Delete masked in
  training tiles; repeat-factor sampling. Defaults taken (not answered): keep the 10 LR scenes > 0.8 m; moderate RFS (t = 0.15).
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
  bilinear to 0.30 m. LR: 208 of 420 degraded to U(0.60, 0.80) the same way (seeded per scene).
- **Delete:** blacked out in train/val tiles; objects with centre in Delete dropped (train 60,441 → 59,957; test kept for scoring, 8,014).
- **Tiles:** 1024 / 819; object kept if ≥ 40 % inside. Train 19,275 tiles → 9,472 with objects + 5,100 background (35 %) = 14,572 unique.
  **RFS** t = 0.15: arty ×3.12, spa ×3.51, tank ×2.01, missile ×1.33, radar ×1.17, camo/logistic ×1.16 → 16,686 tiles per epoch (+2,114 repeats).
  Val (best.pt only): **ALL 4,449 non-empty test tiles** (researcher 20:10: "validate all test scenes", as last time: EXP-2026-10-28-02 validated on all Test116 tiles).
  Final train set after the fix below: 16,709 tiles per epoch (+2,137 repeats).
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
  deterministic; two rebuilds gave the same file list md5), `data/final` rebuilt from scratch. **20:14 final start** of `code/run.sh`.
- Expected (estimate from the first epoch, not measured end-to-end): y8m ~5.5 min/epoch → ~01:45 IST; y11l ~8.5 min/epoch → ~10:15; TTA inference + scoring
  ~1–1.5 h → results ~11:30 IST 2026-11-24 (~15 h total).
- Logs: `logs/pipeline.log`, `logs/train_y8m.log`, `logs/train_y11l.log`, `logs/infer.log`. Reports: `reports/{y8m,y11l,ens_y8m_y11l}.md`.

## Results and failures
- Pending.

## Conclusions and next steps
- Pending. Then: per-class thresholds/selection on these results; optional extra seeds; DINO later.
