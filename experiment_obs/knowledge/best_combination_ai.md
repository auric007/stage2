# How we get the best numbers: per-class model combination (stage 2) — AI-authored

Author: AI agent (Claude). Written 2026-10-27; update this file whenever the combination is re-run.
Built in [EXP-2026-10-27-01](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-27-01-best-combination-v1/record_ai.md).

## 1. Principle (researcher, 2026-10-27)

- **The goal is the best metrics for the product.**
- We are not limited to one model. Different models, from any experiment, may be used for different classes and combined. The models are small YOLOs, so running several is affordable.
- **Every new experiment is a new candidate.** When a run has saved predictions, re-run the combination. If it is better for some class, it replaces the current pick for that class.

## 2. Current best — DEPLOYMENT PICKS (27_10 Test116, labels_simple, IoU 0.3) — 2026-10-29

Built in [EXP-2026-10-28-02](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md).
Candidates: 5 old sets plus 4 sets from m0001's recipe retrained on 27_10 Train359.
Machine-readable spec (set, conf, weight paths per class): `artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/combo_test116.json`.

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 249 | new ensemble, last.pt (new y8m + new y11l) | 0.80 | 48.7 | 53.0 | 50.8 | 132 / 139 / 117 |
| camo | 558 | new ensemble, best.pt (new y8m + new y11l) | 0.90 | 56.1 | 55.7 | 55.9 | 311 / 243 / 247 |
| logistic | 298 | new y11l alone, best.pt | 0.50 | 34.2 | 27.2 | 30.3 | 81 / 156 / 217 |
| missile | 413 | old m0001 ensemble (y8m + y11l) | 0.90 | 71.7 | 50.4 | 59.2 | 208 / 82 / 205 |
| radar | 275 | old m0001 y11l alone | 0.60 | 74.5 | 50.9 | 60.5 | 140 / 48 / 135 |
| smallvehicle | 1242 | old EXP-2026-10-14-13 (y11l fine-tuned on D) | 0.50 | 64.0 | 59.7 | 61.8 | 742 / 417 / 500 |
| tank | 645 | new y8m alone, best.pt | 0.60 | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| vehicle | 3098 | new ensemble, last.pt | 0.90 | 70.2 | 78.3 | 74.0 | 2427 / 1032 / 671 |

| output | MICRO P / R / F1 | MACRO F1 |
|---|---|---|
| **combined (best per class)** | 66.5 / 66.6 / **66.5** | **58.3** |
| **combined + cross-class dedup** (one box per place; the product output) | 68.8 / 63.1 / **65.8** | **57.7** |
| combined, old sets only | F1 65.7 | 57.2 |
| new ensemble alone (best.pt) | F1 65.0 | 57.4 |
| m0001 alone | F1 65.0 | 56.7 |

- With dedup, tank drops 74.2 → 67.7: tank boxes lose to overlapping vehicle boxes, which come from a different set at a higher score. Arty rises 50.8 → 52.1.
- Per-image spread of the combined output is wide (median image F1 66.7, p10 36.4, p90 80.0).
  - Dropping the 5 or 10 worst images moves MICRO by only +0.1 to +1.3 and MACRO by +0.4 to +1.4. The error is spread over the dense scenes, not concentrated in a few bad images.
  - Details: `reports/PER_IMAGE.md`, `reports/per_image_combo.tsv`; per-class tables for each drop scenario in [best_combination_ai_worst_dropped.md](best_combination_ai_worst_dropped.md); radar/missile-specific in [best_combination_ai_worst_dropped_radar_missile.md](best_combination_ai_worst_dropped_radar_missile.md).
- Optimistic: picks and confidences were chosen on Test116 itself, and every model's best.pt was also selected on test tiles. Single seed per new model.
- Scored with labels_simple (2026-10-27 rule), not P04 v2, so not directly comparable with the test_80 table below.
- Tools: `artifacts/EXP-2026-10-28-02-best-recipe-2710/code/compare.py` (picks) and `code/per_image.py` (spec, dedup, per-image breakdown). `combine_best.py` is still wired to test_80 + P04 v2.

**Weights needed for these picks (7 files, 10 model passes per image with TTA sets):**

| weights | path | used for |
|---|---|---|
| new y8m best | `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710/runs/new_y8m/weights/best.pt` | camo (ens), tank |
| new y11l best | `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710/runs/new_y11l/weights/best.pt` | camo (ens), logistic |
| new y8m last | `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710/runs/new_y8m/weights/last.pt` | arty, vehicle (ens) |
| new y11l last | `/sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710/runs/new_y11l/weights/last.pt` | arty, vehicle (ens) |
| m0001 y8m | `/sfs/stage2/models/BEST/prod_y8m_best.pt` | missile (ens) |
| m0001 y11l | `/sfs/stage2/models/BEST/prod_y11l_best.pt` | missile (ens), radar |
| y11l fine-tuned on D | `/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt` | smallvehicle |

Inference per set: `code/engine_v1/infer_test.py --runs <w1>[,<w2>] --scales 1024,1536 --flips none,h --nms 0.4 --conf 0.05 --fuse noisyor --thr 0.05` at 0.30 m GSD.
Then take each class from its set at its confidence, then dedup (radar > missile > arty > logistic, then score; IoU 0.4).
`models/BEST` is unchanged; promoting these picks needs the researcher's decision.

### Previous best (test_80, P04 v2 class mapping, IoU 0.3) — superseded for 27_10 on 2026-10-29

| class        |   GT | model used (prediction set)                | conf ≥ |    P |    R |   F1 | TP / FP / FN     |     |
| ------------ | ---: | ------------------------------------------ | -----: | ---: | ---: | ---: | ---------------- | --- |
| arty         |  162 | EXP-2026-10-14-14 ens: y8m D-frozen + y11l |   0.90 | 42.1 | 37.7 | 39.7 | 61 / 84 / 101    |     |
| camo         |  548 | EXP-2026-10-14-14                          |   0.80 | 50.8 | 49.3 | 50.0 | 270 / 262 / 278  |     |
| logistic     |  241 | EXP-2026-10-14-13 y11l fine-tuned on D     |   0.60 | 26.7 | 30.7 | 28.6 | 74 / 203 / 167   |     |
| missile      |  197 | EXP-2026-10-14-14                          |   0.90 | 66.7 | 61.9 | 64.2 | 122 / 61 / 75    |     |
| radar        |  190 | EXP-2026-10-14-17 y11l original alone      |   0.60 | 71.4 | 50.0 | 58.8 | 95 / 38 / 95     |     |
| smallvehicle |  822 | EXP-2026-10-14-13                          |   0.50 | 62.5 | 49.4 | 55.2 | 406 / 244 / 416  |     |
| tank         |  744 | m0001 ens: y8m + y11l                      |   0.90 | 70.2 | 70.4 | 70.3 | 524 / 222 / 220  |     |
| vehicle      | 2196 | m0001                                      |   0.90 | 68.7 | 77.6 | 72.9 | 1705 / 778 / 491 |     |

| output | MICRO P / R / F1 | MACRO P / R / F1 |
|---|---|---|
| **combined (best per class)** | 63.3 / 63.9 / **63.6** | 57.4 / 53.4 / **55.0** |
| **combined + cross-class dedup** (one box per place; what a product would show) | 66.1 / 59.6 / **62.7** | 59.9 / 50.7 / **54.6** |
| best single prediction set (EXP-2026-10-14-14) | F1 63.4 | F1 54.3 |
| m0001 alone (production) | F1 63.0 | F1 54.0 |
| official P01 scoring, best per class (for reference) | F1 60.2 | F1 49.6 |

**Recall-first variant** (`--objective rp30`): each class at its highest recall with precision ≥ 0.30.
- **Combined:** MICRO R 79.3 / P 32.3; MACRO R 65.5.
- **Per class R:** arty 40.7, camo 63.1, logistic 25.3, missile 72.6, radar 68.4, smallvehicle 68.2, tank 91.9, vehicle 93.5.
- Full table: [report_rp30_ai.md](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-27-01-best-combination-v1/report_rp30_ai.md).

## 3. How the combination is built

1. **Candidates:** every prediction set with saved raw boxes (`c05` = every box with score ≥ 0.05, TTA 1024/1536 × flips).
   - m0001: `artifacts/EXP-2026-09-01-01-final-ensemble-train/predictions_final_c05/labels`;
   - every run: `artifacts/<EXP-ID>/preds/c05/labels`;
   - 21 sets today.
2. **Scoring of each set, class by class** (P04 v2 GT):
   - greedy one-to-one matching at IoU 0.3;
   - predictions in delete regions dropped;
   - confidence sweep 0.05 to 0.90.
3. **Pick per class:** the set and confidence with the highest F1 (or the highest recall at P ≥ 0.30 for the recall-first variant).
4. **Combine:** for each image and class c, keep the class-c boxes of c's picked set with score ≥ c's confidence.
5. **Dedup (product output):** where boxes of different classes overlap (IoU ≥ 0.4), keep one. Priority is radar > missile > arty > logistic, then the higher score (same rule as m0001's deployment).

**Steps 1–5 are one command:**

```bash
cd /sfs/stage2
/sfs/envs/yolo_models/bin/python code/engine_v1/combine_best.py --out artifacts/<EXP-ID>/f1                    # best F1 per class
/sfs/envs/yolo_models/bin/python code/engine_v1/combine_best.py --out artifacts/<EXP-ID>/rp30 --objective rp30 # recall-first
#   --rules none          official P01 mapping instead of P04 v2
#   --rules <file.json>   any other single-class mapping (format: eval/protocols/P04_single_class_remap/rules_v2.json)
#   --iou 0.5             stricter localisation
#   --extra name=<dir>    add a prediction set that is not under artifacts/*/preds/c05/labels
```

**Environment:** `/sfs/envs/yolo_models` lives on /sfs and persists across restarts; no setup step is needed. It takes about 25 s for 21 sets.

**Outputs in `--out`:**

| file | what |
|---|---|
| `REPORT.md` | table above |
| `picks.json` | run, confidence and path per class + totals |
| `leaderboard.tsv` | every set, per-class F1 |
| `per_image.tsv` | image × class GT / TP / FP / FN |
| `preds/combined/labels/<scene>.txt` | the combined boxes |
| `preds/combined_dedup/labels/<scene>.txt` | the combined boxes after dedup |

Format of both prediction folders: `x1 y1 … x4 y4 class score` on the 0.30 m grid (native px × `scale_to_target`). See the [predictions guide](test_predictions_guide_ai.md).

## 4. Running the combination on a new image (product recipe)

**For the 27_10 deployment picks, use the weights table in section 2.** The table below is for the earlier test_80 picks, which needed **4 weight files**:

| weights | path | used in |
|---|---|---|
| y8m (m0001) | `/sfs/stage2/models/BEST/prod_y8m_best.pt` | m0001 |
| y11l (m0001) | `/sfs/stage2/models/BEST/prod_y11l_best.pt` | m0001, EXP-14, EXP-17 |
| y8m fine-tuned on D, frozen backbone | `/sfs/stage2/artifacts/EXP-2026-10-14-11-d-frozen-backbone/runs/ft/weights/best.pt` | EXP-14 |
| y11l fine-tuned on D | `/sfs/stage2/artifacts/EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt` | EXP-13 |

Steps:
1. Resize the image to 0.30 m GSD.
2. Produce the 4 prediction sets with the same TTA ensemble inference used for the runs:
   - `code/engine_v1/infer_test.py --runs <w1>[,<w2>] --thr 0.05`;
   - m0001 = y8m + y11l; EXP-14 = y8m-D-frozen + y11l; EXP-13 = y11l-D; EXP-17 = y11l.
3. Apply the per-class picks (section 2): take each class from its set at its confidence.
4. Dedup.

Cost is 6 model passes per image (2 + 2 + 1 + 1) instead of 2 for m0001.

**Not built yet:**
- a single inference wrapper for new images;
- the stage-2 agent tool (`/sfs/end_to_end/oct_w1/stage_2/tool.py`), which still runs m0001 only.

## 5. Repeating it after future experiments (checklist)

1. The new run must save raw predictions to `artifacts/<EXP-ID>/preds/c05/labels/` (the queue pipeline `code/run_experiment.sh` does this).
2. Allocate an experiment: `bash code/new_experiment.sh best-combination-vN`.
3. Run `combine_best.py` (both objectives).
4. Compare its `REPORT.md` with section 2 here. A class changes its pick only if the new set has a higher F1 for that class.
5. Update section 2 of this file and the weights table in section 4. Note the date and the run that changed.
6. If the class mapping changes (for example, the open ICV → tank question), re-run with the new rules file. Numbers from different mappings are not comparable.

## 6. Caveats

- **Optimistic:** the picks and confidences are chosen on test_80 itself, so the combined number is an upper bound. Before the combination is frozen as the product configuration, choose the picks on a validation split and report test_80 once.
- **Mapping:** P04 v2 is still a draft mapping (D-20261023-01, D-20261026-01). The 15 rules are in `eval/protocols/P04_single_class_remap/rules_v2.json`.
- **Overlaps:** the combined set without dedup allows boxes of different classes on the same object (classes are scored independently). The dedup row is the honest single-output number.
- **Confidences:** they come from a coarse grid (0.05–0.90). A finer grid could add a little.

## 7. History

| date | MICRO / MACRO (combined) | change |
|---|---|---|
| 2026-10-23 | 60.2 / 49.6 | first best-per-class (P01 official), EXP-2026-10-23-01 |
| 2026-10-26 | 63.6 / 55.0 | P04 v2 mapping, EXP-2026-10-26-01/-02 |
| 2026-10-27 | 63.6 / 55.0 (dedup 62.7 / 54.6) | reusable tool `combine_best.py`, EXP-2026-10-27-01 |
| 2026-10-29 | 66.5 / 58.3 (dedup 65.8 / 57.7) | 27_10 Test116 + labels_simple; recipe retrained on Train359 (EXP-2026-10-28-02); deployment picks in section 2 |
