# EXP-2026-11-27-02-cartosat-v41-eval — AI-authored

Author: AI agent (Claude). Generated: 2026-11-27T17:35+05:30.
Human companion: [record.md](record.md).

Status: running.

## Request summary and authorization

2026-11-27 ~17:25 IST, researcher: the CartoSat test set is FINAL (`/sfs/data/lowres_poc/cartosat_test` v4.1). Run inference with our best
models ASAP; the agent chooses the techniques; score with and without the objects at lowres_poc train sites. Scope: inference + scoring only
(no training, no change to protocol, test sets or BEST). Daily log: [2026-11-27_ai.md](../../daily/2026-11-27_ai.md) (TASK-20261127-cartosat-v41-eval).

## Question and design

How do the lowres_poc detectors (v1–v4) and the main-test HYBRID transfer to the final 14-scene CartoSat test (0.6 m, 1,886 scored objects)?
The earlier CartoSat numbers ([EXP-2026-11-25-01](../EXP-2026-11-25-01-lowres-poc-v3/record_ai.md), `reports/carto2611_*`) used the
superseded 8-scene v2 set (offset `268895151` labels, duplicates) and package classes; they are not comparable.

- **Models:** v1 y8m / y11l (EXP-2026-11-23-01), v2 y8m / y11l (EXP-2026-11-24-01), v3 y11l (EXP-2026-11-25-01), v4 y11l / y8m
  (EXP-2026-11-26-02), each `runs/<m>/weights/best.pt`; ensembles v1 / v2 / v4 (y8m + y11l).
- **Inference:** identical to the main test: metadata RGB, per-channel 2–98 % stretch over the whole scene, bilinear to 0.30 m, tiles
  1024 / step 819, TTA 1024/1536 × h-flip, conf 0.05, per-tile noisy-or, scene dedup.
- **Scoring:** the main-test Parent-11 rules (EXP-2026-11-26-02 `score_remap.py`), so the 121-scene main-test thresholds and the main-test
  hybrid apply unchanged. GT: military-support-vehicle, logistic and ICV → vehicle; Tank / SPATracked → tank; radome not scored; doc rule
  (P_Camo = Yes objects also accept camo) and STRICT. GT and predictions centred in Delete are dropped (GT 1,886 = README).
- **Operating points:**
  - MAIN = each model's main-test F1-optimal per-class confs (honest);
  - ORACLE = best F1 per class on CartoSat (optimistic);
  - HYB_MAIN = main-test `hybrid_doc` / `hybrid_strict` picks at their main confs;
  - HYB_ORACLE = per class the best model on CartoSat (optimistic).
- **Scopes:** ALL, and EXCL (the poc-train-site objects of `objects_in_poc_train_sites.tsv` removed, predictions overlapping them dropped);
  IoU 0.3 (primary) and 0.5.

## Actual setup and changes

- Code (not under git): `/sfs/stage2/artifacts/EXP-2026-11-27-02-cartosat-v41-eval/code/` — `infer_carto.py` (tile / raw / set), `run.sh`,
  `score_carto.py`. Env `/sfs/envs/yolo_models`. GPU: 1 × L40S, 3 lanes.
- Tiling: only tiles that intersect the scored area (scene minus Delete) dilated by 300 px at 0.30 m → 7,199 tiles (34,652 for full scenes);
  `268807111` and `268895121` are all Delete (0 tiles). Scoring-neutral: predictions centred in Delete are dropped anyway.
- Reuse: 5,032 tiles (all 7 `CARTOSAT_*` scenes) are byte-identical to the CartoSat v2 tiles of EXP-2026-11-25-01, so the v1–v3 raw
  detections of that run are reused for them; 2,167 tiles (`268895151` re-delivered image: 0 identical, + the new scenes) run fresh. v4 runs all
  7,199. Check: fresh v3 inference on 32 reused tiles = same 10 boxes, scores within 0.002.

## Execution

- 17:2x IST `infer_carto.py tile` → `state/tiles.json`, `data/tiles/` (7,199 JPEG).
- 17:27 IST `code/run.sh` (raw → sets) — running.

## Results and failures

Pending.
