# EXP-2026-11-27-02-cartosat-v41-eval — AI-authored

Author: AI agent (Claude). Generated: 2026-11-27T17:35+05:30.
Human companion: [record.md](record.md).

Status: completed (2026-11-27 ~18:05 IST).

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
- 17:27–17:50 IST `code/run.sh` (raw, 3 GPU lanes → 10 sets: v1/v2/v4 y8m, y11l, ens; v3 y11l). Logs `logs/pipeline.log`, `logs/raw_*.log`, `logs/sets.log`.
- 17:51 IST `score_carto.py` (doc) and `CAMO_ALT=0 score_carto.py` (strict) on all 10 sets → `reports/cartosat_v41_doc.md/.json`, `reports/cartosat_v41_strict.md/.json`.
- Fix + re-score ~17:58 IST: the ORACLE threshold search tied at F1 = 0 for classes without any TP (EXCL scope: missile 2, tank 1, arty 0 GT) and kept conf 0.05 (e.g. 1,411 missile FPs); tie-break now fewest FP (model choice too). MAIN numbers unchanged.

## Results and failures

Doc rule, IoU 0.3, honest operating point (main-test thresholds). MICRO / MACRO over the 7 main-bar classes; strict changes MICRO by ≤ 1.0 (v4 sets
identical); HYB_MAIN strict 49.4 / MACRO 20.1 (its camo pick differs). Full tables (IoU 0.5, strict, all models, per scene): `reports/cartosat_v41_*.md`.

| set | ALL (1,886) MICRO P / R / F1 | MACRO(7) | aerial F1 | class-agnostic F1 | EXCL (876) MICRO F1 | EXCL agnostic F1 |
|---|---|--:|--:|--:|--:|--:|
| v4_ens | 52.4 / 50.7 / **51.6** | 20.3 | 25.0 | **54.5** | **31.6** | **34.3** |
| v4_y11l | 54.6 / 47.6 / 50.9 | **22.7** | **80.0** | 53.6 | 31.1 | 33.7 |
| HYB_MAIN (main-test hybrid) | 52.7 / 47.6 / 50.0 | 22.5 | 25.0 | 52.6 | 30.7 | 33.1 |
| v2_ens | 49.4 / 47.0 / 48.2 | 20.1 | 0 | 51.6 | 30.0 | 33.1 |
| v4_y8m | 47.2 / 50.0 / 48.6 | 19.7 | 0 | 51.6 | 29.2 | 32.2 |
| v1_ens | 47.5 / 47.4 / 47.5 | 21.7 | 0 | 51.5 | 28.9 | 32.3 |
| v3_y11l | 51.0 / 43.8 / 47.1 | 16.9 | 0 | 51.3 | 29.4 | 32.6 |
| HYB_ORACLE (CartoSat-tuned, optimistic) | 48.6 / 54.2 / 51.2 | 31.0 | 80.0 | — | 33.8 | — |

- **Per class (HYB_MAIN, ALL):** vehicle 62.6 (718 / 369 / 490), smallvehicle 28.3, camo 27.4, missile 19.4 (3 / 20), radar 20.0 (2 / 11),
  tank 0 (0 / 8), arty 0 (0 / 6), aerial 25.0 (2 / 6). v4_y11l alone: camo 36.7, smallvehicle 29.2, aerial 80.0 (4 / 6, 0 FP).
- **The main-test hybrid does not transfer:** v4_ens and v4_y11l alone beat it on MICRO; its rare-class picks score 0–3 TP each (≤ 20 GT per class).
- **Poc-train-site objects inflate ALL:** without them (876 GT: vehicle 444, smallvehicle 402, camo 16, rare ≤ 6), MICRO drops from about 51 to 31
  and vehicle F1 from 62.6 to 35.5; FPs are almost unchanged (agnostic 736 → 735), so the drop is all recall on the familiar sites.
- **One scene dominates the FPs:** `Cartosat2_268895131` gives 463 of the 736 class-agnostic FPs (P 8.3 %). 457 of its 502 scored
  vehicle / smallvehicle boxes lie inside the 4 analyst Delete shapes that v4.1 drops on purpose (researcher decision); those areas hold 1 GT
  object. Same check: 268895111 9 / 23, 268895161 9 / 31. Not re-scored with the v4 Delete (researcher's call).
- Per scene (HYB_MAIN agnostic recall): 268895151 74.8 (P 86.8; poc-train site), 42SXB 56.1, 46RCT 54.1, 268895131 51.2, 45RYM 36.9,
  268895111 31.2, 268895161 27.9, 268329211 22.2, 25493 12.0, 50QJL 10.2, 41906 0.0 (22 GT).
- Not comparable with the CartoSat v2 numbers (different scenes, labels, Delete and class rules).

## Supporting evidence

- `/sfs/stage2/artifacts/EXP-2026-11-27-02-cartosat-v41-eval/`: `reports/cartosat_v41_doc.md/.json`, `reports/cartosat_v41_strict.md/.json`,
  `preds/<set>/labels_2x|labels_native/`, `preds/raw/`, `state/tiles.json`, `data/tiles/` (7,199 JPEG, about 3 GB), `logs/`.
- v4 Delete shapes used for the 268895131 check: `/sfs/data/_cleanup_2026-11-02/work/carto27_2/cartosat_test_v4_small_files.tar.gz`.

## Conclusions and next steps

- Best honest CartoSat numbers: v4_ens MICRO F1 51.6 (class-agnostic 54.5); v4_y11l best MACRO(7) 22.7 and the only aerial detector (80.0).
  Rare classes (missile / tank / radar / arty, 6–20 GT each) are near zero for every model and threshold.
- Proposed (not executed): FP review of 268895131 in the fp_viewer (needs restarting on v4.1); a sensitivity score with the analyst Delete
  shapes kept; per-class pick for CartoSat that does not use CartoSat itself (only if the researcher wants a CartoSat-specific operating point).
