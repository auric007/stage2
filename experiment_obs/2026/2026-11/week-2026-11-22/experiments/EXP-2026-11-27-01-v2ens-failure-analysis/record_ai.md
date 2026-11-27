# EXP-2026-11-27-01-v2ens-failure-analysis — AI-authored

Author: AI agent (Claude). Human companion: [record.md](record.md). Daily log: [2026-11-27_ai.md](../../daily/2026-11-27_ai.md).
Builds on: [EXP-2026-11-24-01](../EXP-2026-11-24-01-lowres-poc-v2/record_ai.md) (v2 ensemble = the reference), [EXP-2026-11-26-02](../EXP-2026-11-26-02-lowres-poc-v4-parent11/record_ai.md) (121-scene strict scorer, `reports/radar_missile_recall_all.md`).
Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-11-27-01-v2ens-failure-analysis` — summary `reports/FAILURE_ANALYSIS.md`.

Status: **completed (first pass)** 2026-11-27 15:52 → 16:12 IST. Eval-only: no detector training, no protocol / test-set / deploy change.

## Request summary and authorization
- Researcher 2026-11-27 ~15:50 IST: goes with the **v2 ensemble** for radar / missile at P ≥ 30 / 35 / 40 (decision [D-20261127-01](../../../../../knowledge/decisions_ai.md));
  no training for the next few days; wants visualisation of the systematic failures (which classes are misclassified, top misclassifications with class and
  confidence) and post-processing / analysis ways to lift these numbers.

## Design
- Reference: EXP-2026-11-24-01 `preds/ens_y8m_y11l` (YOLOv8m + YOLO11l, 4 TTA views each, noisy-OR). 121 scenes, Parent-11 STRICT GT, IoU 0.3.
  Operating confs from EXP-2026-11-26-02 `reports/base121_v2_ens__strict.json` (P30, F1) and `rr_v2_ens_P35/P40.json`.
- `code/fa_core.py` re-implements the scorer with per-object outcomes (TP / wrong class / below conf / loose / no box) and per-box sources (TP / on other class /
  unscored object / duplicate / near / background). **Check: TP / FP / n identical to the reports for all 8 classes at P30, P35, P40, F1.**
- Post-processing harness `code/pp.py` (end-to-end re-matching); `refuse.py` rebuilds the ensemble from the cached raw views keeping per-view scores
  (reproduces the reference exactly); `features.py` (24 box features), `rescore.py` (rules + learned rescorer, out-of-fold GroupKFold(5) by scene, 5 shuffles),
  `multi_fuse.py` (adds v1 / v3 / v4 cached raw radar / missile detections), `site_and_remap.py`.

## Results (reference: radar R 63.5 / 63.5 / 62.1, missile 64.3 / 59.4 / 55.7 at P30 / P35 / P40)
- **Errors (P35).** Missed missiles 99 = 63 lost to vehicle (median vehicle conf 0.98), 11 other class, 11 below conf, 14 no box. Missed radars 74 = 33 to vehicle
  (0.90), 5 other, 13 below conf, 3 loose, 20 no box. False missile boxes 264 = 187 on labelled vehicles (123 TransportVehicle, 27 EngineeringVehicle), 35 other
  class, 13 near, 27 background. False radar boxes 214 = 84 background, 50 on vehicles (8 EWS), 40 other class (missile 16, SPA-wheeled 12), 19 dup / loose, 18 near.
- Confusions happen at high confidence: noisy-OR over 8 views saturates (median TP box 1.00, false missile-on-vehicle 0.95).
- Galleries: missed missiles = launchers parked in rows among trucks (MLRS / HQ9 / MissileSystem garrison lots); false missiles = vehicles on SAM-site hardstands.
- SOC: radar recall P35 Pixels 71 % (150) / Site 50 % (30) / Third_Party 30 % (23); missile Pixels 67 % (172) / Nil 50 % / Third_Party 45 % / Site 20 % (20).
- Remap definitions: 15–23 false missile boxes sit on rule-5 objects (empty launchers, transloaders, LD2000, 14 launcher-family objects with Is_Empty `Nil`);
  not counting them → missile 64.8 / 63.5 / 58.2. Radar: 8 on EWS, no change.
- False boxes near real objects (P35): missile 115 / 264 within 200 m of a true missile, 98 in scenes without missiles; radar 93 / 214, 71.
- **Post-processing:** re-fusion of the views (mean / max / votes / per-model) ≤ ±1; rules (vehicle-score penalty, margin, vehicle-park penalty) at best +1 to +2
  on missile at P35/P40 with λ picked on test; train-GT size gate no effect; learned rescorer (out-of-fold by scene) **no gain** (radar 60.5–64.0 at P30, missile
  57.4–62.9). **Adding v3 YOLO11l radar / missile detections with the same noisy-OR rule: radar 66.5 / 65.0 / 63.1, missile 64.8 / 61.1 / 59.4** (all 6 up);
  + v3 + v4: 65.5 / 65.0 / 62.6, 63.9 / 61.1 / 60.2; all 7 models: 66.5 / 65.5 / 63.1, 64.3 / 61.5 / 57.0. Other classes unchanged (v2 boxes).

## Conclusions
- The remaining radar / missile errors are class confusions with vehicles at high confidence, plus ~10 % no-box misses; box-level rescoring cannot separate them
  beyond what the detector score already does. More independent views (another trained model) do help: v2 + v3 y11l is the best no-training variant found.
- Open (researcher's choice): adopt v2 + v3 y11l for radar / missile; whether launcher-family objects with Is_Empty `Nil` should stay vehicle (rule 4 reading).

## Supporting evidence
- Figures `figs/` (confusion_objects / confusion_boxes / child_confusion / top_confusions at P30, P35, P40; rm_failure_modes, rm_scores, rm_recall_by_soc_child),
  galleries `gallery/*_P35_pNN.jpg` (16 sheets), tables `tables/*.tsv`, JSON `reports/postproc_radar_missile.json`, `multi_model_radar_missile.json`, `site_and_remap.json`.
- Environment `/sfs/envs/yolo_models` (sklearn 1.7.2, matplotlib 3.10.8). Code not under git (artifacts dir).

## Follow-up 17:36 IST — analysis reference switched to v2 ens + v3 y11l (radar / missile), D-20261127-02
- Researcher (after the first pass): "yes use this for analysis" for the v2 + v3 YOLO11l radar / missile fusion; also asked which remap rules are applied (answered from
  `14_PARENT11_ONEOFF_INSTANCE_REMAPPING_20261126.md` + v4 `code/remap.py`; table in `reports/FAILURE_ANALYSIS.md` §0).
- Built `preds/v2ens_v3rm/labels_2x` (`code/build_ref.py`: v2 ensemble lines verbatim for other classes + 12-view noisy-OR radar / missile) and scored it with the
  unchanged v4 `score_remap.py` via `code/score_official.py` (reports here; the wrapper reproduces the v4 v2-ensemble report exactly apart from aerial, which no model predicts).
- Official: radar 66.5 (P 30.5) / 65.0 (37.0) / 63.1 (40.8), missile 64.8 (30.1) / 61.1 (35.2) / 59.4 (40.4) at P ≥ 30 / 35 / 40; F1-opt radar 53.2 (77.1), missile 48.0 (57.9);
  any box 76.4 / 79.9. Other classes unchanged. `fa_core.py` reproduces the counts at all op points for both sets.
- `fa_core.py` now takes env `FA_REF` (default `v2v3rm`); v2-only figures / galleries / tables moved to `v2ens_only/`; top-level `figs/`, `gallery/` (P35, 15 sheets),
  `tables/`, `reports/site_and_remap.json` regenerated for the new reference.
- P35 errors (new reference): missed missiles 95 = 60 lost to vehicle (median 0.97), 11 other class, 11 below conf, 13 no box; missed radars 71 = 32 to vehicle (0.92),
  5 other, 13 below conf, 2 loose, 19 no box. False missile boxes 274 = 198 on vehicles (126 TransportVehicle, 32 EngineeringVehicle), 36 other class, 11 near, 27 bg;
  false radar boxes 225 = 89 bg, 56 on vehicles (9 EWS), 42 other class, 22 dup / loose, 16 near.
- v3 adds at P35: radar +6 / −3 (RadarSystem), missile +8 / −4 (5 HQ9); at P40 missile +11 / −2 (7 HQ9). R5 boxes not counted → missile 65.2 / 62.7 / 60.2; radar no change.
- Not re-run on the new set: rules / learned rescorer (they read `state/fused_v2ens.pkl`; no gain on v2).
