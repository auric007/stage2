# Deployment picks with the worst RADAR / MISSILE images dropped — per-class detail (27_10 Test116) — AI-authored

Author: AI agent (Claude). Written 2026-10-29 for deployment, at the researcher's request.
Companions: [best_combination_ai.md §2](best_combination_ai.md) (the picks) and [best_combination_ai_worst_dropped.md](best_combination_ai_worst_dropped.md) (worst images overall).
Built in [EXP-2026-10-28-02](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md).

**Setup**

- Same per-class picks and fixed confidences as best_combination_ai.md §2:
  - radar = old m0001 y11l alone, conf ≥ 0.60;
  - missile = old m0001 ensemble, conf ≥ 0.90.
- Scoring: labels_simple (2026-10-27 rule: radar / missile by Parent Class), IoU 0.3, TTA, delete regions dropped.
- Worst images are ranked on the combined output, using radar and missile counts pooled per image:
  - **lowest radar+missile F1**, among the 101 images with any radar or missile GT or prediction. Ties go to the image with more radar+missile errors.
  - **most radar+missile errors** = largest radar+missile FP + FN.
- **combined** = classes scored independently. **combined + merge** = cross-class dedup, the product output. Radar and missile have top merge priority, so their numbers barely change with the merge.
- Optimistic: picks were chosen on Test116 itself.

## Reading

- **Radar and missile improve:**
  - radar 60.5 → 62–64;
  - missile 59.2 → 64–69 (69.1 after dropping the 10 images with the most radar+missile errors).
- **MICRO is flat, and MACRO goes down** (58.3 → 56.8–57.8), because the same images hold most of the arty detections.
  - Dropping the 5 lowest-radar+missile-F1 images takes arty from 132 to 45 TP and F1 from 50.8 to 30.3.
  - Arty and missile share sites, so removing missile sites removes arty too.
- **What the missed radar / missile objects were predicted as** (10 worst images, `code/rm_confusion.py`, IoU ≥ 0.3 to any box in the deployment output):
  - **Missile, 94 missed:**
    - 61 (65%) overlap a **vehicle** box;
    - 12 overlap an **arty** box, sometimes together with tank or vehicle;
    - 24 (26%) get no box at all;
    - 25 had a missile box from the missile model, but below its 0.90 conf.
  - **Radar, 44 missed:**
    - 19 overlap a vehicle box;
    - 18 get no box;
    - 7 overlap other classes;
    - 12 had a radar box below the 0.60 conf.
  - So the main loss is **missile → vehicle**. In 22MAR07045331 and 22MAR15045922 all 17 and 18 missiles are missed, and 13 and 14 of them come out as vehicle.
  - This fits the 2026-10-27 label rule (missile by Parent Class): vehicles that belong to a missile system are labelled missile, but the models see an ordinary vehicle.
  - Worth reviewing those labels before treating these as model errors.
- **Threshold:** missile's 0.90 conf drops about a quarter of the misses that the model did see. A lower conf would trade precision for recall on these images, but it was not re-tuned here.

Regenerate:

```bash
cd /sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710
/sfs/envs/yolo_models/bin/python3 code/per_image.py   # (re)writes reports/rows_cache.json
python3 code/worst_rm.py                                 # -> reports/WORST_DROPPED_RADAR_MISSILE.md (sections below)
/sfs/envs/yolo_models/bin/python3 code/rm_confusion.py <scene> ...   # missed radar/missile -> predicted class
```

## Summary

| images | MICRO | MACRO | radar F1 | missile F1 | MICRO + merge | MACRO + merge | radar F1 + merge | missile F1 + merge |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| all 116 | 66.5 | 58.3 | 60.5 | 59.2 | 65.8 | 57.7 | 60.5 | 59.2 |
| drop 5 lowest radar+missile F1 | 66.6 | 57.0 | 62.2 | 64.4 | 65.7 | 56.2 | 62.2 | 64.5 |
| drop 10 lowest radar+missile F1 | 65.7 | 57.6 | 63.3 | 65.8 | 64.7 | 56.7 | 63.3 | 65.9 |
| drop 5 with most radar+missile errors | 66.4 | 56.8 | 63.2 | 64.0 | 65.6 | 56.0 | 63.2 | 64.1 |
| drop 10 with most radar+missile errors | 66.8 | 57.8 | 64.1 | 69.1 | 65.9 | 56.9 | 64.1 | 69.3 |

101 of 116 images have any radar/missile GT or prediction (87 with radar GT, 53 with missile GT).

## 10 lowest radar+missile F1 images

| # | image | split | radar GT | radar TP/FP/FN | missile GT | missile TP/FP/FN | radar+missile F1 |
|--:|---|---|--:|---|--:|---|--:|
| 1 | 22MAR07045331-S3DS-200013349499_01_P001 | Test2 | 2 | 0/0/2 | 17 | 0/0/17 | 0.0 |
| 2 | 22MAR15045922-S3DS-200014658599_01_P001 | Test2 | 1 | 0/0/1 | 18 | 0/0/18 | 0.0 |
| 3 | 26d44cd2_DS_PHR1A_202012190354245_FR1_PX_E104N31_0306_00349 | Test2 | 2 | 0/0/2 | 12 | 0/0/12 | 0.0 |
| 4 | 20SEP27061806-S3DS-200011619312_01_P001 | Test2 | 5 | 0/0/5 | 4 | 0/0/4 | 0.0 |
| 5 | 21APR22040621-S3DS-200013119950_01_P001 | Test1 | 3 | 0/0/3 | 6 | 0/0/6 | 0.0 |
| 6 | a5094dad_DS_PHR1B_201911250307191_FR1_PX_E113N23_0107_00341 | Test2 | 2 | 0/1/2 | 4 | 0/0/4 | 0.0 |
| 7 | 25APR24062343-S3DS-200014337763_01_P001 | Test2 | 1 | 0/0/1 | 4 | 0/0/4 | 0.0 |
| 8 | 26JAN17025647-S3DS-200014412603_01_P001 | Test2 | 0 | 0/0/0 | 4 | 0/0/4 | 0.0 |
| 9 | 19SEP02025044-S3DS-200014337621_01_P001 | Test2 | 1 | 0/0/1 | 0 | 0/2/0 | 0.0 |
| 10 | 1eb7aa03_DS_PHR1A_202511290403295_FR1_PX_E102N25_0902_03236 | Test2 | 3 | 0/0/3 | 0 | 0/0/0 | 0.0 |

## 10 images with most radar+missile errors

| # | image | split | radar GT | radar TP/FP/FN | missile GT | missile TP/FP/FN | radar+missile F1 |
|--:|---|---|--:|---|--:|---|--:|
| 1 | 23JUN21043007-S3DS-200013457515_01_P001 | Test2 | 23 | 7/1/16 | 13 | 2/0/11 | 39.1 |
| 2 | 19MAY14023939-S3DS-200014337623_01_P001 | Test2 | 1 | 1/1/0 | 16 | 12/18/4 | 53.1 |
| 3 | 22MAR07045331-S3DS-200013349499_01_P001 | Test2 | 2 | 0/0/2 | 17 | 0/0/17 | 0.0 |
| 4 | 22MAR15045922-S3DS-200014658599_01_P001 | Test2 | 1 | 0/0/1 | 18 | 0/0/18 | 0.0 |
| 5 | 22OCT24030421-S3DS-200011896352_01_P001 | Test1 | 19 | 5/0/14 | 0 | 0/1/0 | 40.0 |
| 6 | 26d44cd2_DS_PHR1A_202012190354245_FR1_PX_E104N31_0306_00349 | Test2 | 2 | 0/0/2 | 12 | 0/0/12 | 0.0 |
| 7 | 24APR01051503-S3DS-200012320748_01_P001 | Test1 | 0 | 0/0/0 | 17 | 5/2/12 | 41.7 |
| 8 | 22AUG10022400-S3DS-200014337620_01_P001 | Test2 | 2 | 1/1/1 | 15 | 5/2/10 | 46.2 |
| 9 | 22MAY17025959-S3DS-200012307431_01_P001 | Test1 | 0 | 0/3/0 | 8 | 1/3/7 | 13.3 |
| 10 | 24APR01051503-S3DS-200012288324_01_P001 | Test1 | 0 | 0/0/0 | 18 | 6/0/12 | 50.0 |

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

## drop 5 lowest radar+missile F1 (111 images)

Dropped: `22MAR07045331-S3DS-200013349499_01_P001` (Test2), `22MAR15045922-S3DS-200014658599_01_P001` (Test2), `26d44cd2_DS_PHR1A_202012190354245_FR1_PX_E104N31_0306_00349` (Test2), `20SEP27061806-S3DS-200011619312_01_P001` (Test2), `21APR22040621-S3DS-200013119950_01_P001` (Test1)

**combined** — MICRO P 66.8 / R 66.4 / F1 66.6 · MACRO F1 57.0

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 31.9 | 28.8 | 30.3 | 45 / 96 / 111 |
| camo | 556 | new_ens | 0.90 | 56.1 | 55.9 | 56.0 | 311 / 243 / 245 |
| logistic | 296 | new_y11l_alone | 0.50 | 35.5 | 27.4 | 30.9 | 81 / 147 / 215 |
| missile | 356 | old_m0001_ens | 0.90 | 71.7 | 58.4 | 64.4 | 208 / 82 / 148 |
| radar | 262 | old_y11l_alone | 0.60 | 74.5 | 53.4 | 62.2 | 140 / 48 / 122 |
| smallvehicle | 1216 | old_exp13_y11l_ftD | 0.50 | 64.4 | 60.1 | 62.2 | 731 / 404 / 485 |
| tank | 643 | new_y8m_alone | 0.60 | 78.5 | 73.7 | 76.0 | 474 / 130 / 169 |
| vehicle | 2854 | new_ens_last | 0.90 | 70.1 | 77.9 | 73.8 | 2222 / 946 / 632 |

**combined + merge (cross-class dedup, product output)** — MICRO P 68.9 / R 62.8 / F1 65.7 · MACRO F1 56.2

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 35.2 | 28.8 | 31.7 | 45 / 83 / 111 |
| camo | 556 | new_ens | 0.90 | 59.1 | 54.1 | 56.5 | 301 / 208 / 255 |
| logistic | 296 | new_y11l_alone | 0.50 | 35.7 | 27.4 | 31.0 | 81 / 146 / 215 |
| missile | 356 | old_m0001_ens | 0.90 | 72.4 | 58.1 | 64.5 | 207 / 79 / 149 |
| radar | 262 | old_y11l_alone | 0.60 | 74.5 | 53.4 | 62.2 | 140 / 48 / 122 |
| smallvehicle | 1216 | old_exp13_y11l_ftD | 0.50 | 65.3 | 58.1 | 61.5 | 706 / 375 / 510 |
| tank | 643 | new_y8m_alone | 0.60 | 82.8 | 57.5 | 67.9 | 370 / 77 / 273 |
| vehicle | 2854 | new_ens_last | 0.90 | 73.2 | 74.7 | 74.0 | 2133 / 780 / 721 |

## drop 10 lowest radar+missile F1 (106 images)

Dropped: `22MAR07045331-S3DS-200013349499_01_P001` (Test2), `22MAR15045922-S3DS-200014658599_01_P001` (Test2), `26d44cd2_DS_PHR1A_202012190354245_FR1_PX_E104N31_0306_00349` (Test2), `20SEP27061806-S3DS-200011619312_01_P001` (Test2), `21APR22040621-S3DS-200013119950_01_P001` (Test1), `a5094dad_DS_PHR1B_201911250307191_FR1_PX_E113N23_0107_00341` (Test2), `25APR24062343-S3DS-200014337763_01_P001` (Test2), `26JAN17025647-S3DS-200014412603_01_P001` (Test2), `19SEP02025044-S3DS-200014337621_01_P001` (Test2), `1eb7aa03_DS_PHR1A_202511290403295_FR1_PX_E102N25_0902_03236` (Test2)

**combined** — MICRO P 65.8 / R 65.6 / F1 65.7 · MACRO F1 57.6

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 36.3 | 28.8 | 32.1 | 45 / 79 / 111 |
| camo | 556 | new_ens | 0.90 | 57.9 | 55.9 | 56.9 | 311 / 226 / 245 |
| logistic | 286 | new_y11l_alone | 0.50 | 38.8 | 28.3 | 32.7 | 81 / 128 / 205 |
| missile | 344 | old_m0001_ens | 0.90 | 72.2 | 60.5 | 65.8 | 208 / 80 / 136 |
| radar | 255 | old_y11l_alone | 0.60 | 74.9 | 54.9 | 63.3 | 140 / 47 / 115 |
| smallvehicle | 1110 | old_exp13_y11l_ftD | 0.50 | 64.1 | 59.1 | 61.5 | 656 / 368 / 454 |
| tank | 643 | new_y8m_alone | 0.60 | 79.8 | 73.7 | 76.6 | 474 / 120 / 169 |
| vehicle | 2375 | new_ens_last | 0.90 | 67.0 | 77.6 | 71.9 | 1842 / 909 / 533 |

**combined + merge (cross-class dedup, product output)** — MICRO P 67.9 / R 61.8 / F1 64.7 · MACRO F1 56.7

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 40.2 | 28.8 | 33.6 | 45 / 67 / 111 |
| camo | 556 | new_ens | 0.90 | 60.3 | 54.1 | 57.1 | 301 / 198 / 255 |
| logistic | 286 | new_y11l_alone | 0.50 | 38.9 | 28.3 | 32.8 | 81 / 127 / 205 |
| missile | 344 | old_m0001_ens | 0.90 | 72.9 | 60.2 | 65.9 | 207 / 77 / 137 |
| radar | 255 | old_y11l_alone | 0.60 | 74.9 | 54.9 | 63.3 | 140 / 47 / 115 |
| smallvehicle | 1110 | old_exp13_y11l_ftD | 0.50 | 65.0 | 56.8 | 60.6 | 631 / 340 / 479 |
| tank | 643 | new_y8m_alone | 0.60 | 83.1 | 57.5 | 68.0 | 370 / 75 / 273 |
| vehicle | 2375 | new_ens_last | 0.90 | 70.3 | 74.2 | 72.2 | 1762 / 743 / 613 |

## drop 5 with most radar+missile errors (111 images)

Dropped: `23JUN21043007-S3DS-200013457515_01_P001` (Test2), `19MAY14023939-S3DS-200014337623_01_P001` (Test2), `22MAR07045331-S3DS-200013349499_01_P001` (Test2), `22MAR15045922-S3DS-200014658599_01_P001` (Test2), `22OCT24030421-S3DS-200011896352_01_P001` (Test1)

**combined** — MICRO P 67.1 / R 65.7 / F1 66.4 · MACRO F1 56.8

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 35.2 | 28.8 | 31.7 | 45 / 83 / 111 |
| camo | 523 | new_ens | 0.90 | 55.0 | 54.5 | 54.8 | 285 / 233 / 238 |
| logistic | 261 | new_y11l_alone | 0.50 | 32.4 | 26.4 | 29.1 | 69 / 144 / 192 |
| missile | 349 | old_m0001_ens | 0.90 | 75.5 | 55.6 | 64.0 | 194 / 63 / 155 |
| radar | 229 | old_y11l_alone | 0.60 | 73.4 | 55.5 | 63.2 | 127 / 46 / 102 |
| smallvehicle | 1211 | old_exp13_y11l_ftD | 0.50 | 65.3 | 60.0 | 62.5 | 726 / 386 / 485 |
| tank | 643 | new_y8m_alone | 0.60 | 78.6 | 73.7 | 76.1 | 474 / 129 / 169 |
| vehicle | 2732 | new_ens_last | 0.90 | 70.4 | 76.6 | 73.4 | 2092 / 879 / 640 |

**combined + merge (cross-class dedup, product output)** — MICRO P 69.2 / R 62.3 / F1 65.6 · MACRO F1 56.0

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 156 | new_ens_last | 0.80 | 37.5 | 28.8 | 32.6 | 45 / 75 / 111 |
| camo | 523 | new_ens | 0.90 | 58.0 | 52.8 | 55.3 | 276 / 200 / 247 |
| logistic | 261 | new_y11l_alone | 0.50 | 32.5 | 26.4 | 29.2 | 69 / 143 / 192 |
| missile | 349 | old_m0001_ens | 0.90 | 76.3 | 55.3 | 64.1 | 193 / 60 / 156 |
| radar | 229 | old_y11l_alone | 0.60 | 73.4 | 55.5 | 63.2 | 127 / 46 / 102 |
| smallvehicle | 1211 | old_exp13_y11l_ftD | 0.50 | 66.2 | 58.1 | 61.9 | 704 / 359 / 507 |
| tank | 643 | new_y8m_alone | 0.60 | 83.0 | 57.5 | 68.0 | 370 / 76 / 273 |
| vehicle | 2732 | new_ens_last | 0.90 | 73.3 | 73.9 | 73.6 | 2019 / 736 / 713 |

## drop 10 with most radar+missile errors (106 images)

Dropped: `23JUN21043007-S3DS-200013457515_01_P001` (Test2), `19MAY14023939-S3DS-200014337623_01_P001` (Test2), `22MAR07045331-S3DS-200013349499_01_P001` (Test2), `22MAR15045922-S3DS-200014658599_01_P001` (Test2), `22OCT24030421-S3DS-200011896352_01_P001` (Test1), `26d44cd2_DS_PHR1A_202012190354245_FR1_PX_E104N31_0306_00349` (Test2), `24APR01051503-S3DS-200012320748_01_P001` (Test1), `22AUG10022400-S3DS-200014337620_01_P001` (Test2), `22MAY17025959-S3DS-200012307431_01_P001` (Test1), `24APR01051503-S3DS-200012288324_01_P001` (Test1)

**combined** — MICRO P 67.6 / R 66.0 / F1 66.8 · MACRO F1 57.8

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 155 | new_ens_last | 0.80 | 36.0 | 29.0 | 32.1 | 45 / 80 / 110 |
| camo | 513 | new_ens | 0.90 | 55.4 | 55.6 | 55.5 | 285 / 229 / 228 |
| logistic | 247 | new_y11l_alone | 0.50 | 31.6 | 26.3 | 28.7 | 65 / 141 / 182 |
| missile | 279 | old_m0001_ens | 0.90 | 76.0 | 63.4 | 69.1 | 177 / 56 / 102 |
| radar | 225 | old_y11l_alone | 0.60 | 75.0 | 56.0 | 64.1 | 126 / 42 / 99 |
| smallvehicle | 1207 | old_exp13_y11l_ftD | 0.50 | 65.7 | 60.1 | 62.7 | 725 / 379 / 482 |
| tank | 642 | new_y8m_alone | 0.60 | 79.1 | 73.8 | 76.4 | 474 / 125 / 168 |
| vehicle | 2558 | new_ens_last | 0.90 | 71.1 | 76.2 | 73.6 | 1950 / 792 / 608 |

**combined + merge (cross-class dedup, product output)** — MICRO P 69.5 / R 62.5 / F1 65.9 · MACRO F1 56.9

| class | GT | prediction set | conf ≥ | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 155 | new_ens_last | 0.80 | 38.5 | 29.0 | 33.1 | 45 / 72 / 110 |
| camo | 513 | new_ens | 0.90 | 58.5 | 53.8 | 56.0 | 276 / 196 / 237 |
| logistic | 247 | new_y11l_alone | 0.50 | 31.7 | 26.3 | 28.8 | 65 / 140 / 182 |
| missile | 279 | old_m0001_ens | 0.90 | 76.9 | 63.1 | 69.3 | 176 / 53 / 103 |
| radar | 225 | old_y11l_alone | 0.60 | 75.0 | 56.0 | 64.1 | 126 / 42 / 99 |
| smallvehicle | 1207 | old_exp13_y11l_ftD | 0.50 | 66.6 | 58.2 | 62.2 | 703 / 352 / 504 |
| tank | 642 | new_y8m_alone | 0.60 | 83.0 | 57.6 | 68.0 | 370 / 76 / 272 |
| vehicle | 2558 | new_ens_last | 0.90 | 73.9 | 73.6 | 73.7 | 1883 / 666 / 675 |

