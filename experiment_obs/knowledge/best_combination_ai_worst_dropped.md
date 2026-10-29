# Deployment picks with the worst images dropped — per-class detail (27_10 Test116) — AI-authored

Author: AI agent (Claude). Written 2026-10-29 for deployment, at the researcher's request.
Companion to [best_combination_ai.md §2](best_combination_ai.md). Built in [EXP-2026-10-28-02](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md).

**Setup**

- Same per-class picks as best_combination_ai.md §2: each class comes from its picked prediction set, at its picked conf.
- Confidences are held fixed; nothing is re-tuned after dropping images.
- Scoring: labels_simple (2026-10-27 rule), IoU 0.3, delete regions dropped, TTA 1024/1536 × none/h, noisy-or, NMS 0.4.
- **combined** = classes scored independently, so a place can carry boxes of two classes.
- **combined + merge** = cross-class dedup: IoU ≥ 0.4 keeps one box, priority radar > missile > arty > logistic, then score. This is the product output.
- Worst-image rankings are both computed on the combined output:
  - **lowest-F1** = lowest per-image micro F1, among images with any GT or prediction;
  - **most errors** = largest FP + FN.
- Optimistic: picks and confidences were chosen on Test116 itself, and best.pt of each model was selected on test tiles.

## Summary

| images                   | MICRO | MACRO | MICRO + merge | MACRO + merge |
| ------------------------ | ----: | ----: | ------------: | ------------: |
| all 116                  |  66.5 |  58.3 |          65.8 |          57.7 |
| drop 5 lowest-F1         |  66.8 |  58.7 |          66.0 |          58.0 |
| drop 10 lowest-F1        |  67.8 |  59.3 |          67.0 |          58.7 |
| drop 5 with most errors  |  66.6 |  59.7 |          66.4 |          60.2 |
| drop 10 with most errors |  66.6 |  58.8 |          66.2 |          58.9 |

**Reading**

- Median per-image F1 is 66.7 (p10 36.4, p25 51.2, p75 74.8, p90 80.0).
- Lowest-F1 images are mostly small scenes (1–10 GT) with a few FPs, plus `23MAY19062258…` (121 of 190 GT are missed smallvehicles; worth checking the labels and GSD).
- Most-error images are dense scenes (150–460 GT) that also hold many TPs. Dropping 10 of them lowers arty to 35–38, because most arty TPs are in them.
- The merge mainly costs tank: 74.2 → 67.7, as tank boxes lose to overlapping vehicle boxes from another set with a higher score.

Regenerate: `cd /sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710 && /sfs/envs/yolo_models/bin/python3 code/per_image.py`.
That writes `reports/WORST_DROPPED.md` (the sections below), `reports/PER_IMAGE.md` and `reports/per_image_combo.tsv`.

## all 116 (116 images)

**combined** — MICRO P 66.5 / R 66.6 / F1 66.5 · MACRO F1 58.3

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 249 | new_ens_last | 0.80 | 48.7 | 53.0 | 50.8 | 132 / 139 / 117 |
| camo | 558 | new_ens | 0.90 | 56.1 | 55.7 | 55.9 | 311 / 243 / 247 |
| logistic | 298 | new_y11l_alone | 0.50 | 34.2 | 27.2 | 30.3 | 81 / 156 / 217 |
| missile | 413 | old_m0001_ens | 0.90 | 71.7 | 50.4 | 59.2 | 208 / 82 / 205 |
| radar | 275 | old_y11l_alone | 0.60 | 74.5 | 50.9 | 60.5 | 140 / 48 / 135 |
| smallvehicle | 1242 | old_exp13_y11l_ftD | 0.50 | 64.0 | 59.7 | 61.8 | 742 / 417 / 500 |
| tank | 645 | new_y8m_alone | 0.60 | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| vehicle | 3098 | new_ens_last | 0.90 | 70.2 | 78.3 | 74.0 | 2427 / 1032 / 671 |

**combined + merge (cross-class dedup, product output)** — MICRO P 68.8 / R 63.1 / F1 65.8 · MACRO F1 57.7

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 249 | new_ens_last | 0.80 | 51.2 | 53.0 | 52.1 | 132 / 126 / 117 |
| camo | 558 | new_ens | 0.90 | 59.1 | 53.9 | 56.4 | 301 / 208 / 257 |
| logistic | 298 | new_y11l_alone | 0.50 | 34.3 | 27.2 | 30.3 | 81 / 155 / 217 |
| missile | 413 | old_m0001_ens | 0.90 | 72.4 | 50.1 | 59.2 | 207 / 79 / 206 |
| radar | 275 | old_y11l_alone | 0.60 | 74.5 | 50.9 | 60.5 | 140 / 48 / 135 |
| smallvehicle | 1242 | old_exp13_y11l_ftD | 0.50 | 64.8 | 57.6 | 61.0 | 715 / 388 / 527 |
| tank | 645 | new_y8m_alone | 0.60 | 82.6 | 57.4 | 67.7 | 370 / 78 / 275 |
| vehicle | 3098 | new_ens_last | 0.90 | 73.1 | 75.1 | 74.1 | 2328 / 858 / 770 |

## drop 5 lowest-F1 (111 images)

Dropped: `23JAN27044234-P3DS-200012483678_01_P001` (Test2), `1eb7aa03_DS_PHR1A_202511290403295_FR1_PX_E102N25_0902_03236` (Test2), `24NOV02023241-S3DS-200014631155_01_P001` (Test2), `24MAR01041407-S3DS-200013190860_01_P001` (Test1), `19SEP29022142-S3DS-200013073804_01_P001` (Test1)

**combined** — MICRO P 66.7 / R 66.9 / F1 66.8 · MACRO F1 58.7

| class        |   GT | prediction set     | conf ≥ |    P |    R |   F1 | TP / FP / FN      |
| ------------ | ---: | ------------------ | -----: | ---: | ---: | ---: | ----------------- |
| arty         |  241 | new_ens_last       |   0.80 | 48.7 | 54.8 | 51.6 | 132 / 139 / 109   |
| camo         |  554 | new_ens            |   0.90 | 56.4 | 56.1 | 56.3 | 311 / 240 / 243   |
| logistic     |  298 | new_y11l_alone     |   0.50 | 35.1 | 27.2 | 30.6 | 81 / 150 / 217    |
| missile      |  413 | old_m0001_ens      |   0.90 | 71.7 | 50.4 | 59.2 | 208 / 82 / 205    |
| radar        |  268 | old_y11l_alone     |   0.60 | 75.1 | 51.9 | 61.4 | 139 / 46 / 129    |
| smallvehicle | 1239 | old_exp13_y11l_ftD |   0.50 | 64.7 | 59.9 | 62.2 | 742 / 404 / 497   |
| tank         |  645 | new_y8m_alone      |   0.60 | 74.7 | 73.6 | 74.2 | 475 / 161 / 170   |
| vehicle      | 3094 | new_ens_last       |   0.90 | 70.2 | 78.4 | 74.1 | 2426 / 1029 / 668 |

**combined + merge (cross-class dedup, product output)** — MICRO P 69.1 / R 63.3 / F1 66.0 · MACRO F1 58.0

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 241 | new_ens_last | 0.80 | 51.2 | 54.8 | 52.9 | 132 / 126 / 109 |
| camo | 554 | new_ens | 0.90 | 59.5 | 54.3 | 56.8 | 301 / 205 / 253 |
| logistic | 298 | new_y11l_alone | 0.50 | 35.2 | 27.2 | 30.7 | 81 / 149 / 217 |
| missile | 413 | old_m0001_ens | 0.90 | 72.4 | 50.1 | 59.2 | 207 / 79 / 206 |
| radar | 268 | old_y11l_alone | 0.60 | 75.1 | 51.9 | 61.4 | 139 / 46 / 129 |
| smallvehicle | 1239 | old_exp13_y11l_ftD | 0.50 | 65.6 | 57.7 | 61.4 | 715 / 375 / 524 |
| tank | 645 | new_y8m_alone | 0.60 | 82.6 | 57.4 | 67.7 | 370 / 78 / 275 |
| vehicle | 3094 | new_ens_last | 0.90 | 73.1 | 75.2 | 74.2 | 2327 / 855 / 767 |

## drop 10 lowest-F1 (106 images)

Dropped: `23JAN27044234-P3DS-200012483678_01_P001` (Test2), `1eb7aa03_DS_PHR1A_202511290403295_FR1_PX_E102N25_0902_03236` (Test2), `24NOV02023241-S3DS-200014631155_01_P001` (Test2), `24MAR01041407-S3DS-200013190860_01_P001` (Test1), `19SEP29022142-S3DS-200013073804_01_P001` (Test1), `24MAR31023121-S3DS-200013073812_01_P001` (Test1), `23MAY19062258-S3DS-200011619589_01_P001` (Test2), `22SEP12053238-S3DS-200014159522_01_P001` (Test2), `20FEB09030243-S3DS-200014588197_01_P001` (Test2), `25JAN15024759-S3DS-200014337605_01_P001` (Test2)

**combined** — MICRO P 66.8 / R 68.7 / F1 67.8 · MACRO F1 59.3

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 237 | new_ens_last | 0.80 | 48.9 | 55.7 | 52.1 | 132 / 138 / 105 |
| camo | 554 | new_ens | 0.90 | 56.9 | 56.1 | 56.5 | 311 / 236 / 243 |
| logistic | 288 | new_y11l_alone | 0.50 | 34.9 | 27.8 | 30.9 | 80 / 149 / 208 |
| missile | 408 | old_m0001_ens | 0.90 | 71.7 | 51.0 | 59.6 | 208 / 82 / 200 |
| radar | 258 | old_y11l_alone | 0.60 | 74.7 | 51.6 | 61.0 | 133 / 45 / 125 |
| smallvehicle | 1100 | old_exp13_y11l_ftD | 0.50 | 65.0 | 66.5 | 65.7 | 731 / 393 / 369 |
| tank | 645 | new_y8m_alone | 0.60 | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| vehicle | 3014 | new_ens_last | 0.90 | 70.3 | 79.6 | 74.7 | 2400 / 1016 / 614 |

**combined + merge (cross-class dedup, product output)** — MICRO P 69.2 / R 65.0 / F1 67.0 · MACRO F1 58.7

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 237 | new_ens_last | 0.80 | 51.4 | 55.7 | 53.4 | 132 / 125 / 105 |
| camo | 554 | new_ens | 0.90 | 60.0 | 54.3 | 57.0 | 301 / 201 / 253 |
| logistic | 288 | new_y11l_alone | 0.50 | 35.1 | 27.8 | 31.0 | 80 / 148 / 208 |
| missile | 408 | old_m0001_ens | 0.90 | 72.4 | 50.7 | 59.7 | 207 / 79 / 201 |
| radar | 258 | old_y11l_alone | 0.60 | 74.7 | 51.6 | 61.0 | 133 / 45 / 125 |
| smallvehicle | 1100 | old_exp13_y11l_ftD | 0.50 | 65.9 | 64.0 | 64.9 | 704 / 364 / 396 |
| tank | 645 | new_y8m_alone | 0.60 | 82.6 | 57.4 | 67.7 | 370 / 78 / 275 |
| vehicle | 3014 | new_ens_last | 0.90 | 73.1 | 76.4 | 74.7 | 2302 / 845 / 712 |

## drop 5 with most errors (111 images)

Dropped: `22OCT21045409-S3DS-200012483671_01_P001` (Test2), `22JUL01052041-S3DS-200013802003_01_P001` (Test2), `19SEP02025044-S3DS-200014337621_01_P001` (Test2), `23JAN30025247-S3DS-200012319991_01_P001` (Test2), `22FEB25024136-S3DS-200012371966_01_P001` (Test1)

**combined** — MICRO P 67.2 / R 66.0 / F1 66.6 · MACRO F1 59.7

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 216 | new_ens_last | 0.80 | 59.5 | 56.5 | 58.0 | 122 / 83 / 94 |
| camo | 324 | new_ens | 0.90 | 65.8 | 48.8 | 56.0 | 158 / 82 / 166 |
| logistic | 270 | new_y11l_alone | 0.50 | 38.9 | 30.0 | 33.9 | 81 / 127 / 189 |
| missile | 413 | old_m0001_ens | 0.90 | 72.2 | 50.4 | 59.3 | 208 / 80 / 205 |
| radar | 273 | old_y11l_alone | 0.60 | 75.3 | 51.3 | 61.0 | 140 / 46 / 133 |
| smallvehicle | 1077 | old_exp13_y11l_ftD | 0.50 | 63.4 | 59.6 | 61.4 | 642 / 371 / 435 |
| tank | 266 | new_y8m_alone | 0.60 | 70.4 | 77.8 | 73.9 | 207 / 87 / 59 |
| vehicle | 2393 | new_ens_last | 0.90 | 70.0 | 79.1 | 74.3 | 1894 / 811 / 499 |

**combined + merge (cross-class dedup, product output)** — MICRO P 69.8 / R 63.2 / F1 66.4 · MACRO F1 60.2

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 216 | new_ens_last | 0.80 | 63.2 | 56.5 | 59.7 | 122 / 71 / 94 |
| camo | 324 | new_ens | 0.90 | 69.9 | 46.6 | 55.9 | 151 / 65 / 173 |
| logistic | 270 | new_y11l_alone | 0.50 | 39.1 | 30.0 | 34.0 | 81 / 126 / 189 |
| missile | 413 | old_m0001_ens | 0.90 | 72.9 | 50.1 | 59.4 | 207 / 77 / 206 |
| radar | 273 | old_y11l_alone | 0.60 | 75.3 | 51.3 | 61.0 | 140 / 46 / 133 |
| smallvehicle | 1077 | old_exp13_y11l_ftD | 0.50 | 63.9 | 57.1 | 60.3 | 615 / 348 / 462 |
| tank | 266 | new_y8m_alone | 0.60 | 84.3 | 70.7 | 76.9 | 188 / 35 / 78 |
| vehicle | 2393 | new_ens_last | 0.90 | 73.1 | 75.4 | 74.2 | 1805 / 665 / 588 |

## drop 10 with most errors (106 images)

Dropped: `22OCT21045409-S3DS-200012483671_01_P001` (Test2), `22JUL01052041-S3DS-200013802003_01_P001` (Test2), `19SEP02025044-S3DS-200014337621_01_P001` (Test2), `23JAN30025247-S3DS-200012319991_01_P001` (Test2), `22FEB25024136-S3DS-200012371966_01_P001` (Test1), `23MAY19062258-S3DS-200011619589_01_P001` (Test2), `22MAR15045922-S3DS-200014658599_01_P001` (Test2), `20OCT06054149-S3DS-200012493568_01_P001` (Test2), `22MAR07045331-S3DS-200013349499_01_P001` (Test2), `23APR08030249-S3DS-200014649611_01_P001` (Test2)

**combined** — MICRO P 66.7 / R 66.6 / F1 66.6 · MACRO F1 58.8

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 123 | new_ens_last | 0.80 | 46.7 | 28.5 | 35.4 | 35 / 40 / 88 |
| camo | 273 | new_ens | 0.90 | 65.4 | 56.8 | 60.8 | 155 / 82 / 118 |
| logistic | 228 | new_y11l_alone | 0.50 | 39.3 | 31.6 | 35.0 | 72 / 111 / 156 |
| missile | 356 | old_m0001_ens | 0.90 | 71.3 | 55.1 | 62.1 | 196 / 79 / 160 |
| radar | 252 | old_y11l_alone | 0.60 | 74.6 | 51.2 | 60.7 | 129 / 44 / 123 |
| smallvehicle | 845 | old_exp13_y11l_ftD | 0.50 | 62.2 | 65.9 | 64.0 | 557 / 339 / 288 |
| tank | 256 | new_y8m_alone | 0.60 | 79.2 | 80.5 | 79.8 | 206 / 54 / 50 |
| vehicle | 1916 | new_ens_last | 0.90 | 69.0 | 77.2 | 72.8 | 1479 / 666 / 437 |

**combined + merge (cross-class dedup, product output)** — MICRO P 68.9 / R 63.6 / F1 66.2 · MACRO F1 58.9

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 123 | new_ens_last | 0.80 | 55.6 | 28.5 | 37.6 | 35 / 28 / 88 |
| camo | 273 | new_ens | 0.90 | 69.5 | 54.2 | 60.9 | 148 / 65 / 125 |
| logistic | 228 | new_y11l_alone | 0.50 | 39.6 | 31.6 | 35.1 | 72 / 110 / 156 |
| missile | 356 | old_m0001_ens | 0.90 | 72.0 | 54.8 | 62.2 | 195 / 76 / 161 |
| radar | 252 | old_y11l_alone | 0.60 | 74.6 | 51.2 | 60.7 | 129 / 44 / 123 |
| smallvehicle | 845 | old_exp13_y11l_ftD | 0.50 | 62.2 | 63.1 | 62.6 | 533 / 324 / 312 |
| tank | 256 | new_y8m_alone | 0.60 | 85.5 | 73.4 | 79.0 | 188 / 32 / 68 |
| vehicle | 1916 | new_ens_last | 0.90 | 72.1 | 73.3 | 72.7 | 1404 / 542 / 512 |

