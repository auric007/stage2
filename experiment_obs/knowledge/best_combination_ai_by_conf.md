# Best model per class × confidence (0.1–0.9) on all 116 Test116 images — AI-authored

Author: AI agent (Claude). Written 2026-10-29 at the researcher's request: for every class and every conf 0.1, 0.2 … 0.9, which model gives the best precision, the best recall and the best F1, over every stage-2 experiment so far.
Companions: [best_combination_ai.md §2](best_combination_ai.md) (deployment picks), [worst images dropped](best_combination_ai_worst_dropped.md), [radar/missile worst dropped](best_combination_ai_worst_dropped_radar_missile.md).
Built in [EXP-2026-10-28-02](../2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-28-02-best-recipe-2710/record_ai.md).

**What is covered: 25 prediction sets, every stage-2 detector trained so far.**

- m0001 (y8m, y11l, ensemble).
- All EXP-2026-10-14/15 relabel and fine-tune variants (b, c, d, r, control, D-frozen, y11l-D, y11l-DR, low-lr), alone and in their ensembles.
- The recipe retrained on 27_10 Train359 (EXP-2026-10-28-02): y8m, y11l, ensemble best.pt and last.pt.
- Not included:
  - the subset-study proxies (EXP-2026-10-28-01: short 15-epoch y8m models, no booster, much weaker);
  - methods that change the GT mapping (P04 remap, attribute merge rules; EXP-2026-10-12-01, 10-23-02, 10-26-01). Their numbers are on a different label definition.

**Scoring:**

- Every set is inferred on all 116 Test116 images with the same TTA pipeline: 1024/1536 × none/h, noisy-or, NMS 0.4, boxes ≥ 0.05.
- labels_simple (2026-10-27 rule: no P_Camo merge, radar/missile by Parent Class), IoU 0.3, delete regions dropped.
- Classes are scored independently (no cross-class merge).

**How to read the tables:**

- At each conf, three winners:
  - **best P** — ties go to more TP;
  - **best R** — ties go to higher P;
  - **best F1** — ties go to higher P.
- Best P at a high conf can be won by a conservative model with low recall; check its TP count.
- All picks are made on Test116 itself, so they are optimistic.

## Candidate update to the deployment picks (not yet applied)

With the 16 extra sets, two classes get a new best-F1 winner. Both gains are tiny and within seed noise:

| class | current pick (§2) | new best |
|---|---|---|
| smallvehicle | EXP-13 y11l-D @ 0.5: F1 61.8 | **EXP-18 y11l control fine-tune @ 0.4: F1 62.2** |
| vehicle | new ensemble last.pt @ 0.9: F1 74.0 | **EXP-07 ens (y8m fine-tuned on relabel c + m0001 y11l) @ 0.9: F1 74.1** |

| deployment combination | MICRO | MACRO | MICRO + merge | MACRO + merge |
|---|--:|--:|--:|--:|
| current §2 picks (9 sets) | 66.5 | 58.3 | 65.8 | 57.7 |
| best over 25 sets | 66.5 | 58.4 | 65.8 | **58.2** |

- After the merge, MACRO gains +0.5. Most of it is tank: 67.7 → 72.0, because fewer vehicle boxes now overrule tank boxes.
- The candidate needs 8 weight files: the EXP-13 file drops out, and the EXP-18 and EXP-02 fine-tuned weights come in.
- Files:
  - `artifacts/EXP-2026-10-28-02-best-recipe-2710/reports/combo_test116_all25.json`;
  - per-image and drop-worst tables in `reports/PER_IMAGE_all25.md` and `reports/WORST_DROPPED_all25.md`.
- §2 stays unchanged until the researcher decides.

Regenerate:

```bash
cd /sfs/stage2/artifacts/EXP-2026-10-28-02-best-recipe-2710
python3 code/best_by_conf.py reports/BEST_BY_CONF.md                          # this table (all reports/*.test116.json)
ALLSETS=1 /sfs/envs/yolo_models/bin/python3 code/per_image.py                  # combination + merge over all sets -> *_all25
bash code/infer_set.sh <name> <w1.pt[,w2.pt]>                                  # add a new set (TTA inference + scoring)
```

25 prediction sets, all on the 116 Test116 images. TTA 1024/1536 × none/h, noisy-or, NMS 0.4; labels_simple (2026-10-27 rule); IoU 0.3; delete regions dropped; classes scored independently (no cross-class merge).

## Overview — best F1 per class at any conf

| class | GT | set | conf | P | R | F1 | TP / FP / FN |
|---|--:|---|--:|--:|--:|--:|---|
| arty | 249 | new_ens_last | 0.8 | 48.7 | 53.0 | 50.8 | 132 / 139 / 117 |
| camo | 558 | new_ens | 0.9 | 56.1 | 55.7 | 55.9 | 311 / 243 / 247 |
| logistic | 298 | new_y11l_alone | 0.5 | 34.2 | 27.2 | 30.3 | 81 / 156 / 217 |
| missile | 413 | old_m0001_ens | 0.9 | 71.7 | 50.4 | 59.2 | 208 / 82 / 205 |
| radar | 275 | old_y11l_alone | 0.6 | 74.5 | 50.9 | 60.5 | 140 / 48 / 135 |
| smallvehicle | 1242 | old_exp18_y11lft_control | 0.4 | 61.5 | 63.0 | 62.2 | 782 / 490 / 460 |
| tank | 645 | new_y8m_alone | 0.6 | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| vehicle | 3098 | old_exp07_ens_y8mft_c_y11l | 0.9 | 71.9 | 76.4 | 74.1 | 2368 / 926 / 730 |

## arty (GT 249)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | new_y11l_alone | 26.5 | 61.8 | 37.1 | 154 / 428 / 95 |
| 0.1 | best R | new_ens_last | 21.2 | 68.7 | 32.4 | 171 / 635 / 78 |
| 0.1 | best F1 | new_y11l_alone | 26.5 | 61.8 | 37.1 | 154 / 428 / 95 |
| 0.2 | best P | new_y11l_alone | 32.0 | 59.8 | 41.7 | 149 / 316 / 100 |
| 0.2 | best R | new_ens_last | 25.5 | 66.3 | 36.8 | 165 / 482 / 84 |
| 0.2 | best F1 | new_y11l_alone | 32.0 | 59.8 | 41.7 | 149 / 316 / 100 |
| 0.3 | best P | new_y8m_alone | 37.3 | 57.0 | 45.1 | 142 / 239 / 107 |
| 0.3 | best R | new_ens_last | 28.6 | 63.9 | 39.6 | 159 / 396 / 90 |
| 0.3 | best F1 | new_y8m_alone | 37.3 | 57.0 | 45.1 | 142 / 239 / 107 |
| 0.4 | best P | new_y8m_alone | 44.3 | 53.4 | 48.5 | 133 / 167 / 116 |
| 0.4 | best R | new_ens_last | 32.1 | 62.7 | 42.4 | 156 / 330 / 93 |
| 0.4 | best F1 | new_y8m_alone | 44.3 | 53.4 | 48.5 | 133 / 167 / 116 |
| 0.5 | best P | new_y8m_alone | 50.4 | 49.4 | 49.9 | 123 / 121 / 126 |
| 0.5 | best R | new_ens_last | 35.4 | 61.8 | 45.0 | 154 / 281 / 95 |
| 0.5 | best F1 | new_y8m_alone | 50.4 | 49.4 | 49.9 | 123 / 121 / 126 |
| 0.6 | best P | new_y8m_alone | 57.2 | 43.0 | 49.1 | 107 / 80 / 142 |
| 0.6 | best R | new_ens_last | 39.8 | 59.4 | 47.7 | 148 / 224 / 101 |
| 0.6 | best F1 | new_ens | 44.2 | 55.4 | 49.2 | 138 / 174 / 111 |
| 0.7 | best P | new_y8m_alone | 62.6 | 41.0 | 49.5 | 102 / 61 / 147 |
| 0.7 | best R | new_ens_last | 44.0 | 57.4 | 49.8 | 143 / 182 / 106 |
| 0.7 | best F1 | new_ens | 49.8 | 51.4 | 50.6 | 128 / 129 / 121 |
| 0.8 | best P | new_y8m_alone | 67.2 | 34.5 | 45.6 | 86 / 42 / 163 |
| 0.8 | best R | new_ens_last | 48.7 | 53.0 | 50.8 | 132 / 139 / 117 |
| 0.8 | best F1 | new_ens_last | 48.7 | 53.0 | 50.8 | 132 / 139 / 117 |
| 0.9 | best P | new_y8m_alone | 70.1 | 27.3 | 39.3 | 68 / 29 / 181 |
| 0.9 | best R | new_ens_last | 52.9 | 47.8 | 50.2 | 119 / 106 / 130 |
| 0.9 | best F1 | new_ens_last | 52.9 | 47.8 | 50.2 | 119 / 106 / 130 |

## camo (GT 558)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | new_y11l_alone | 28.5 | 67.7 | 40.1 | 378 / 949 / 180 |
| 0.1 | best R | old_exp19_y11lft_dr | 19.4 | 76.2 | 30.9 | 425 / 1768 / 133 |
| 0.1 | best F1 | new_y11l_alone | 28.5 | 67.7 | 40.1 | 378 / 949 / 180 |
| 0.2 | best P | new_y11l_alone | 35.2 | 66.3 | 46.0 | 370 / 680 / 188 |
| 0.2 | best R | old_exp19_y11lft_dr | 23.8 | 72.9 | 35.8 | 407 / 1306 / 151 |
| 0.2 | best F1 | new_y11l_alone | 35.2 | 66.3 | 46.0 | 370 / 680 / 188 |
| 0.3 | best P | new_y11l_alone | 39.7 | 62.9 | 48.6 | 351 / 534 / 207 |
| 0.3 | best R | new_ens | 28.0 | 70.6 | 40.1 | 394 / 1013 / 164 |
| 0.3 | best F1 | new_y11l_alone | 39.7 | 62.9 | 48.6 | 351 / 534 / 207 |
| 0.4 | best P | new_y11l_alone | 45.3 | 59.9 | 51.5 | 334 / 404 / 224 |
| 0.4 | best R | new_ens | 31.7 | 69.2 | 43.5 | 386 / 832 / 172 |
| 0.4 | best F1 | new_y11l_alone | 45.3 | 59.9 | 51.5 | 334 / 404 / 224 |
| 0.5 | best P | new_y11l_alone | 49.8 | 57.3 | 53.3 | 320 / 322 / 238 |
| 0.5 | best R | new_ens | 35.1 | 67.2 | 46.2 | 375 / 692 / 183 |
| 0.5 | best F1 | new_y11l_alone | 49.8 | 57.3 | 53.3 | 320 / 322 / 238 |
| 0.6 | best P | new_y11l_alone | 55.1 | 55.2 | 55.1 | 308 / 251 / 250 |
| 0.6 | best R | new_ens_last | 38.4 | 66.5 | 48.7 | 371 / 594 / 187 |
| 0.6 | best F1 | new_y11l_alone | 55.1 | 55.2 | 55.1 | 308 / 251 / 250 |
| 0.7 | best P | new_y11l_alone | 60.2 | 52.0 | 55.8 | 290 / 192 / 268 |
| 0.7 | best R | new_ens_last | 41.4 | 64.2 | 50.3 | 358 / 507 / 200 |
| 0.7 | best F1 | new_y11l_alone | 60.2 | 52.0 | 55.8 | 290 / 192 / 268 |
| 0.8 | best P | new_y11l_alone | 67.4 | 47.1 | 55.5 | 263 / 127 / 295 |
| 0.8 | best R | new_ens_last | 45.8 | 62.0 | 52.7 | 346 / 410 / 212 |
| 0.8 | best F1 | new_y11l_alone | 67.4 | 47.1 | 55.5 | 263 / 127 / 295 |
| 0.9 | best P | new_y11l_alone | 74.0 | 40.9 | 52.7 | 228 / 80 / 330 |
| 0.9 | best R | new_ens_last | 52.1 | 58.2 | 55.0 | 325 / 299 / 233 |
| 0.9 | best F1 | new_ens | 56.1 | 55.7 | 55.9 | 311 / 243 / 247 |

## logistic (GT 298)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | old_exp19_y11lft_dr | 16.2 | 54.4 | 25.0 | 162 / 838 / 136 |
| 0.1 | best R | old_exp14_ens_y8mDfrozen_y11l | 8.6 | 60.1 | 15.0 | 179 / 1914 / 119 |
| 0.1 | best F1 | old_exp19_y11lft_dr | 16.2 | 54.4 | 25.0 | 162 / 838 / 136 |
| 0.2 | best P | old_exp03_y8mft_d | 19.7 | 35.2 | 25.3 | 105 / 428 / 193 |
| 0.2 | best R | old_exp1501_ens_y11l_y11ld | 13.8 | 56.4 | 22.2 | 168 / 1046 / 130 |
| 0.2 | best F1 | old_exp19_y11lft_dr | 18.6 | 44.0 | 26.1 | 131 / 573 / 167 |
| 0.3 | best P | old_exp12_y8mft_r | 24.3 | 29.9 | 26.8 | 89 / 277 / 209 |
| 0.3 | best R | old_exp1501_ens_y11l_y11ld | 15.3 | 50.7 | 23.5 | 151 / 835 / 147 |
| 0.3 | best F1 | new_y11l_alone | 23.8 | 35.2 | 28.4 | 105 / 337 / 193 |
| 0.4 | best P | new_y11l_alone | 29.8 | 30.2 | 30.0 | 90 / 212 / 208 |
| 0.4 | best R | old_exp1501_ens_y11l_y11ld | 17.0 | 47.0 | 25.0 | 140 / 684 / 158 |
| 0.4 | best F1 | new_y11l_alone | 29.8 | 30.2 | 30.0 | 90 / 212 / 208 |
| 0.5 | best P | new_y11l_alone | 34.2 | 27.2 | 30.3 | 81 / 156 / 217 |
| 0.5 | best R | new_ens_last | 19.7 | 42.6 | 26.9 | 127 / 519 / 171 |
| 0.5 | best F1 | new_y11l_alone | 34.2 | 27.2 | 30.3 | 81 / 156 / 217 |
| 0.6 | best P | old_exp03_y8mft_d | 37.2 | 17.1 | 23.4 | 51 / 86 / 247 |
| 0.6 | best R | new_ens_last | 22.5 | 39.3 | 28.6 | 117 / 403 / 181 |
| 0.6 | best F1 | new_ens_last | 22.5 | 39.3 | 28.6 | 117 / 403 / 181 |
| 0.7 | best P | old_exp19_y11lft_dr | 40.8 | 21.5 | 28.1 | 64 / 93 / 234 |
| 0.7 | best R | old_exp14_ens_y8mDfrozen_y11l | 23.6 | 35.2 | 28.3 | 105 / 340 / 193 |
| 0.7 | best F1 | new_ens_last | 25.9 | 34.2 | 29.5 | 102 / 292 / 196 |
| 0.8 | best P | old_exp19_y11lft_dr | 46.9 | 15.4 | 23.2 | 46 / 52 / 252 |
| 0.8 | best R | old_exp1501_ens_y11l_y11ld | 26.1 | 32.2 | 28.8 | 96 / 272 / 202 |
| 0.8 | best F1 | old_exp06_ens_y8mft_b_y11l | 33.9 | 25.5 | 29.1 | 76 / 148 / 222 |
| 0.9 | best P | old_exp19_y11lft_dr | 62.5 | 8.4 | 14.8 | 25 / 15 / 273 |
| 0.9 | best R | old_exp1501_ens_y11l_y11ld | 29.7 | 24.5 | 26.8 | 73 / 173 / 225 |
| 0.9 | best F1 | old_exp14_ens_y8mDfrozen_y11l | 34.1 | 23.8 | 28.1 | 71 / 137 / 227 |

## missile (GT 413)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | old_exp19_y11lft_dr | 45.5 | 55.9 | 50.2 | 231 / 277 / 182 |
| 0.1 | best R | old_exp15_ens_y8mft_r_y11l | 23.0 | 70.7 | 34.8 | 292 / 975 / 121 |
| 0.1 | best F1 | old_exp1502_y11lft_d_lowlr | 44.3 | 58.1 | 50.3 | 240 / 302 / 173 |
| 0.2 | best P | old_exp19_y11lft_dr | 53.2 | 55.0 | 54.0 | 227 / 200 / 186 |
| 0.2 | best R | old_exp15_ens_y8mft_r_y11l | 28.3 | 69.2 | 40.2 | 286 / 723 / 127 |
| 0.2 | best F1 | old_exp1502_y11lft_d_lowlr | 52.4 | 55.9 | 54.1 | 231 / 210 / 182 |
| 0.3 | best P | old_exp19_y11lft_dr | 60.7 | 52.1 | 56.1 | 215 / 139 / 198 |
| 0.3 | best R | old_exp15_ens_y8mft_r_y11l | 32.8 | 66.1 | 43.8 | 273 / 560 / 140 |
| 0.3 | best F1 | old_exp19_y11lft_dr | 60.7 | 52.1 | 56.1 | 215 / 139 / 198 |
| 0.4 | best P | old_exp19_y11lft_dr | 65.2 | 47.2 | 54.8 | 195 / 104 / 218 |
| 0.4 | best R | old_exp09_ens_y8mft_control_y11l | 37.4 | 64.4 | 47.3 | 266 / 446 / 147 |
| 0.4 | best F1 | old_exp1502_y11lft_d_lowlr | 63.8 | 50.8 | 56.6 | 210 / 119 / 203 |
| 0.5 | best P | old_exp19_y11lft_dr | 69.6 | 44.3 | 54.1 | 183 / 80 / 230 |
| 0.5 | best R | old_exp09_ens_y8mft_control_y11l | 41.5 | 62.5 | 49.9 | 258 / 363 / 155 |
| 0.5 | best F1 | old_y8m_alone | 58.9 | 55.4 | 57.1 | 229 / 160 / 184 |
| 0.6 | best P | old_exp19_y11lft_dr | 74.2 | 41.2 | 53.0 | 170 / 59 / 243 |
| 0.6 | best R | old_exp09_ens_y8mft_control_y11l | 46.3 | 61.5 | 52.8 | 254 / 295 / 159 |
| 0.6 | best F1 | old_exp14_ens_y8mDfrozen_y11l | 58.6 | 56.9 | 57.7 | 235 / 166 / 178 |
| 0.7 | best P | old_exp19_y11lft_dr | 81.2 | 37.8 | 51.6 | 156 / 36 / 257 |
| 0.7 | best R | old_exp15_ens_y8mft_r_y11l | 50.8 | 59.8 | 54.9 | 247 / 239 / 166 |
| 0.7 | best F1 | old_exp14_ens_y8mDfrozen_y11l | 63.6 | 52.8 | 57.7 | 218 / 125 / 195 |
| 0.8 | best P | old_exp19_y11lft_dr | 84.8 | 33.9 | 48.4 | 140 / 25 / 273 |
| 0.8 | best R | old_exp15_ens_y8mft_r_y11l | 55.9 | 57.1 | 56.5 | 236 / 186 / 177 |
| 0.8 | best F1 | old_exp14_ens_y8mDfrozen_y11l | 70.0 | 50.4 | 58.6 | 208 / 89 / 205 |
| 0.9 | best P | old_exp19_y11lft_dr | 91.0 | 29.3 | 44.3 | 121 / 12 / 292 |
| 0.9 | best R | old_exp15_ens_y8mft_r_y11l | 63.4 | 53.8 | 58.2 | 222 / 128 / 191 |
| 0.9 | best F1 | old_m0001_ens | 71.7 | 50.4 | 59.2 | 208 / 82 / 205 |

## radar (GT 275)

| conf | criterion | set                              |    P |    R |   F1 | TP / FP / FN    |
| ---: | --------- | -------------------------------- | ---: | ---: | ---: | --------------- |
|  0.1 | best P    | old_exp12_y8mft_r                | 41.4 | 57.1 | 48.0 | 157 / 222 / 118 |
|  0.1 | best R    | new_ens_last                     | 29.3 | 74.5 | 42.1 | 205 / 495 / 70  |
|  0.1 | best F1   | new_y8m_alone                    | 39.8 | 67.6 | 50.1 | 186 / 281 / 89  |
|  0.2 | best P    | old_exp12_y8mft_r                | 50.7 | 53.1 | 51.9 | 146 / 142 / 129 |
|  0.2 | best R    | new_ens_last                     | 35.3 | 70.2 | 47.0 | 193 / 353 / 82  |
|  0.2 | best F1   | new_y11l_alone                   | 50.2 | 58.5 | 54.0 | 161 / 160 / 114 |
|  0.3 | best P    | old_exp12_y8mft_r                | 60.3 | 50.9 | 55.2 | 140 / 92 / 135  |
|  0.3 | best R    | new_ens_last                     | 42.0 | 69.1 | 52.3 | 190 / 262 / 85  |
|  0.3 | best F1   | old_y11l_alone                   | 56.4 | 59.3 | 57.8 | 163 / 126 / 112 |
|  0.4 | best P    | old_exp12_y8mft_r                | 66.7 | 46.5 | 54.8 | 128 / 64 / 147  |
|  0.4 | best R    | new_ens_last                     | 47.4 | 65.8 | 55.1 | 181 / 201 / 94  |
|  0.4 | best F1   | old_y11l_alone                   | 60.8 | 56.4 | 58.5 | 155 / 100 / 120 |
|  0.5 | best P    | old_exp12_y8mft_r                | 70.6 | 43.6 | 53.9 | 120 / 50 / 155  |
|  0.5 | best R    | new_ens_last                     | 51.9 | 63.6 | 57.2 | 175 / 162 / 100 |
|  0.5 | best F1   | new_ens                          | 60.4 | 60.0 | 60.2 | 165 / 108 / 110 |
|  0.6 | best P    | old_y11l_alone                   | 74.5 | 50.9 | 60.5 | 140 / 48 / 135  |
|  0.6 | best R    | new_ens_last                     | 56.2 | 60.7 | 58.4 | 167 / 130 / 108 |
|  0.6 | best F1   | old_y11l_alone                   | 74.5 | 50.9 | 60.5 | 140 / 48 / 135  |
|  0.7 | best P    | new_y11l_alone                   | 80.0 | 40.7 | 54.0 | 112 / 28 / 163  |
|  0.7 | best R    | new_ens_last                     | 60.5 | 57.8 | 59.1 | 159 / 104 / 116 |
|  0.7 | best F1   | old_exp09_ens_y8mft_control_y11l | 67.1 | 53.5 | 59.5 | 147 / 72 / 128  |
|  0.8 | best P    | new_y11l_alone                   | 87.3 | 34.9 | 49.9 | 96 / 14 / 179   |
|  0.8 | best R    | new_ens_last                     | 65.4 | 54.9 | 59.7 | 151 / 80 / 124  |
|  0.8 | best F1   | new_ens_last                     | 65.4 | 54.9 | 59.7 | 151 / 80 / 124  |
|  0.9 | best P    | new_y11l_alone                   | 89.0 | 26.5 | 40.9 | 73 / 9 / 202    |
|  0.9 | best R    | new_ens_last                     | 72.0 | 50.5 | 59.4 | 139 / 54 / 136  |
|  0.9 | best F1   | new_ens_last                     | 72.0 | 50.5 | 59.4 | 139 / 54 / 136  |

## smallvehicle (GT 1242)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | old_exp19_y11lft_dr | 46.9 | 62.6 | 53.6 | 778 / 881 / 464 |
| 0.1 | best R | new_ens_last | 33.3 | 81.4 | 47.2 | 1011 / 2029 / 231 |
| 0.1 | best F1 | new_y11l_alone | 45.6 | 71.3 | 55.7 | 886 / 1055 / 356 |
| 0.2 | best P | old_exp19_y11lft_dr | 56.1 | 57.2 | 56.7 | 711 / 557 / 531 |
| 0.2 | best R | old_exp14_ens_y8mDfrozen_y11l | 38.7 | 78.9 | 51.9 | 980 / 1551 / 262 |
| 0.2 | best F1 | old_exp18_y11lft_control | 50.7 | 70.9 | 59.1 | 881 / 857 / 361 |
| 0.3 | best P | old_exp19_y11lft_dr | 62.4 | 52.5 | 57.0 | 652 / 393 / 590 |
| 0.3 | best R | old_exp14_ens_y8mDfrozen_y11l | 44.8 | 75.2 | 56.2 | 934 / 1150 / 308 |
| 0.3 | best F1 | old_exp18_y11lft_control | 56.6 | 67.1 | 61.4 | 833 / 638 / 409 |
| 0.4 | best P | old_exp19_y11lft_dr | 67.6 | 47.0 | 55.5 | 584 / 280 / 658 |
| 0.4 | best R | new_ens_last | 49.1 | 71.9 | 58.3 | 893 / 927 / 349 |
| 0.4 | best F1 | old_exp18_y11lft_control | 61.5 | 63.0 | 62.2 | 782 / 490 / 460 |
| 0.5 | best P | old_exp19_y11lft_dr | 71.1 | 41.6 | 52.5 | 517 / 210 / 725 |
| 0.5 | best R | new_ens_last | 53.2 | 68.0 | 59.7 | 845 / 742 / 397 |
| 0.5 | best F1 | old_exp18_y11lft_control | 65.8 | 58.6 | 62.0 | 728 / 378 / 514 |
| 0.6 | best P | old_exp19_y11lft_dr | 74.1 | 36.3 | 48.7 | 451 / 158 / 791 |
| 0.6 | best R | new_ens_last | 56.0 | 64.4 | 59.9 | 800 / 629 / 442 |
| 0.6 | best F1 | old_exp16_ens_y8md_y11ld | 60.3 | 62.9 | 61.5 | 781 / 515 / 461 |
| 0.7 | best P | old_exp19_y11lft_dr | 77.9 | 30.7 | 44.0 | 381 / 108 / 861 |
| 0.7 | best R | new_ens_last | 60.0 | 59.3 | 59.6 | 736 / 490 / 506 |
| 0.7 | best F1 | old_exp1501_ens_y11l_y11ld | 64.3 | 58.6 | 61.3 | 728 / 405 / 514 |
| 0.8 | best P | old_exp19_y11lft_dr | 79.2 | 23.9 | 36.7 | 297 / 78 / 945 |
| 0.8 | best R | old_exp1501_ens_y11l_y11ld | 69.2 | 54.5 | 61.0 | 677 / 301 / 565 |
| 0.8 | best F1 | old_exp1501_ens_y11l_y11ld | 69.2 | 54.5 | 61.0 | 677 / 301 / 565 |
| 0.9 | best P | old_y11l_alone | 86.0 | 19.3 | 31.6 | 240 / 39 / 1002 |
| 0.9 | best R | new_ens_last | 69.6 | 47.7 | 56.6 | 592 / 259 / 650 |
| 0.9 | best F1 | new_ens_last | 69.6 | 47.7 | 56.6 | 592 / 259 / 650 |

## tank (GT 645)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | old_y11l_alone | 48.4 | 88.5 | 62.6 | 571 / 609 / 74 |
| 0.1 | best R | old_m0001_ens | 36.8 | 92.6 | 52.7 | 597 / 1024 / 48 |
| 0.1 | best F1 | old_y11l_alone | 48.4 | 88.5 | 62.6 | 571 / 609 / 74 |
| 0.2 | best P | new_y8m_alone | 55.4 | 84.5 | 66.9 | 545 / 439 / 100 |
| 0.2 | best R | new_ens | 42.3 | 91.3 | 57.8 | 589 / 804 / 56 |
| 0.2 | best F1 | new_y8m_alone | 55.4 | 84.5 | 66.9 | 545 / 439 / 100 |
| 0.3 | best P | new_y8m_alone | 60.4 | 82.6 | 69.8 | 533 / 349 / 112 |
| 0.3 | best R | new_ens | 47.0 | 90.5 | 61.9 | 584 / 659 / 61 |
| 0.3 | best F1 | new_y8m_alone | 60.4 | 82.6 | 69.8 | 533 / 349 / 112 |
| 0.4 | best P | new_y8m_alone | 66.0 | 79.5 | 72.2 | 513 / 264 / 132 |
| 0.4 | best R | new_ens | 51.2 | 89.3 | 65.0 | 576 / 550 / 69 |
| 0.4 | best F1 | new_y8m_alone | 66.0 | 79.5 | 72.2 | 513 / 264 / 132 |
| 0.5 | best P | new_y8m_alone | 70.6 | 76.3 | 73.3 | 492 / 205 / 153 |
| 0.5 | best R | new_ens | 54.5 | 87.9 | 67.3 | 567 / 473 / 78 |
| 0.5 | best F1 | new_y8m_alone | 70.6 | 76.3 | 73.3 | 492 / 205 / 153 |
| 0.6 | best P | new_y8m_alone | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| 0.6 | best R | new_ens | 58.2 | 86.7 | 69.6 | 559 / 402 / 86 |
| 0.6 | best F1 | new_y8m_alone | 74.7 | 73.6 | 74.2 | 475 / 161 / 170 |
| 0.7 | best P | new_y8m_alone | 78.2 | 67.9 | 72.7 | 438 / 122 / 207 |
| 0.7 | best R | new_ens | 61.6 | 85.1 | 71.5 | 549 / 342 / 96 |
| 0.7 | best F1 | old_exp11_y8m_dfrozen | 76.9 | 70.2 | 73.4 | 453 / 136 / 192 |
| 0.8 | best P | new_y8m_alone | 81.9 | 60.5 | 69.6 | 390 / 86 / 255 |
| 0.8 | best R | new_ens | 65.2 | 82.3 | 72.8 | 531 / 283 / 114 |
| 0.8 | best F1 | old_exp19_y11lft_dr | 72.5 | 73.3 | 72.9 | 473 / 179 / 172 |
| 0.9 | best P | new_y8m_alone | 86.0 | 45.9 | 59.9 | 296 / 48 / 349 |
| 0.9 | best R | new_ens | 70.9 | 76.7 | 73.7 | 495 / 203 / 150 |
| 0.9 | best F1 | new_ens | 70.9 | 76.7 | 73.7 | 495 / 203 / 150 |

## vehicle (GT 3098)

| conf | criterion | set | P | R | F1 | TP / FP / FN |
|--:|---|---|--:|--:|--:|---|
| 0.1 | best P | new_y11l_alone | 45.2 | 86.9 | 59.5 | 2693 / 3259 / 405 |
| 0.1 | best R | old_exp14_ens_y8mDfrozen_y11l | 31.1 | 92.5 | 46.6 | 2865 / 6334 / 233 |
| 0.1 | best F1 | new_y11l_alone | 45.2 | 86.9 | 59.5 | 2693 / 3259 / 405 |
| 0.2 | best P | new_y11l_alone | 52.4 | 83.7 | 64.4 | 2594 / 2358 / 504 |
| 0.2 | best R | old_exp16_ens_y8md_y11ld | 40.0 | 91.1 | 55.6 | 2821 / 4225 / 277 |
| 0.2 | best F1 | new_y11l_alone | 52.4 | 83.7 | 64.4 | 2594 / 2358 / 504 |
| 0.3 | best P | new_y11l_alone | 57.8 | 81.3 | 67.6 | 2519 / 1839 / 579 |
| 0.3 | best R | old_exp14_ens_y8mDfrozen_y11l | 43.3 | 90.0 | 58.4 | 2788 / 3654 / 310 |
| 0.3 | best F1 | new_y11l_alone | 57.8 | 81.3 | 67.6 | 2519 / 1839 / 579 |
| 0.4 | best P | new_y11l_alone | 61.9 | 79.1 | 69.4 | 2450 / 1510 / 648 |
| 0.4 | best R | old_exp14_ens_y8mDfrozen_y11l | 47.6 | 88.9 | 62.0 | 2755 / 3028 / 343 |
| 0.4 | best F1 | new_y11l_alone | 61.9 | 79.1 | 69.4 | 2450 / 1510 / 648 |
| 0.5 | best P | new_y11l_alone | 65.2 | 75.8 | 70.1 | 2348 / 1252 / 750 |
| 0.5 | best R | old_exp14_ens_y8mDfrozen_y11l | 51.5 | 87.6 | 64.9 | 2715 / 2554 / 383 |
| 0.5 | best F1 | old_exp04_y8mft_control | 64.2 | 78.7 | 70.7 | 2437 / 1356 / 661 |
| 0.6 | best P | new_y11l_alone | 69.1 | 72.8 | 70.9 | 2255 / 1008 / 843 |
| 0.6 | best R | old_exp14_ens_y8mDfrozen_y11l | 55.5 | 86.2 | 67.5 | 2672 / 2143 / 426 |
| 0.6 | best F1 | old_exp12_y8mft_r | 67.9 | 76.9 | 72.1 | 2383 / 1129 / 715 |
| 0.7 | best P | new_y11l_alone | 73.7 | 69.2 | 71.4 | 2144 / 765 / 954 |
| 0.7 | best R | old_exp14_ens_y8mDfrozen_y11l | 59.6 | 84.8 | 70.0 | 2626 / 1783 / 472 |
| 0.7 | best F1 | old_y11l_alone | 70.6 | 76.2 | 73.3 | 2360 / 982 / 738 |
| 0.8 | best P | new_y11l_alone | 77.8 | 63.1 | 69.7 | 1954 / 557 / 1144 |
| 0.8 | best R | old_exp14_ens_y8mDfrozen_y11l | 63.7 | 82.4 | 71.9 | 2553 / 1455 / 545 |
| 0.8 | best F1 | old_y11l_alone | 74.7 | 72.4 | 73.5 | 2243 / 759 / 855 |
| 0.9 | best P | new_y11l_alone | 83.4 | 54.8 | 66.1 | 1698 / 339 / 1400 |
| 0.9 | best R | old_exp14_ens_y8mDfrozen_y11l | 69.4 | 78.4 | 73.6 | 2430 / 1072 / 668 |
| 0.9 | best F1 | old_exp07_ens_y8mft_c_y11l | 71.9 | 76.4 | 74.1 | 2368 / 926 / 730 |

## Prediction sets (name → weights; ensembles fused with noisy-or)

| set | MICRO F1 | MACRO F1 | weights |
|---|--:|--:|---|
| new_ens | 65.0 | 57.4 | `EXP-2026-10-28-02-best-recipe-2710/runs/new_y8m/weights/best.pt` + `EXP-2026-10-28-02-best-recipe-2710/runs/new_y11l/weights/best.pt` |
| new_ens_last | 65.1 | 57.0 | `EXP-2026-10-28-02-best-recipe-2710/runs/new_y8m/weights/last.pt` + `EXP-2026-10-28-02-best-recipe-2710/runs/new_y11l/weights/last.pt` |
| old_m0001_ens | 65.0 | 56.7 | `models/BEST/prod_y8m_best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp14_ens_y8mDfrozen_y11l | 65.3 | 56.7 | `EXP-2026-10-14-11-d-frozen-backbone/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp15_ens_y8mft_r_y11l | 65.0 | 56.4 | `EXP-2026-10-14-12-radar-r-stowed-vehicle/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp09_ens_y8mft_control_y11l | 65.2 | 56.4 | `EXP-2026-10-14-04-control-finetune-a-unchanged/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp07_ens_y8mft_c_y11l | 65.4 | 56.3 | `EXP-2026-10-14-02-relabel-c-unloaded-vehicle/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp06_ens_y8mft_b_y11l | 65.3 | 56.2 | `EXP-2026-10-14-01-relabel-b-nonlauncher-vehicle/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| old_exp08_ens_y8mft_d_y11l | 65.3 | 56.2 | `EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt` + `models/BEST/prod_y11l_best.pt` |
| new_y11l_alone | 63.7 | 56.1 | `EXP-2026-10-28-02-best-recipe-2710/runs/new_y11l/weights/best.pt` |
| old_exp16_ens_y8md_y11ld | 64.9 | 56.1 | `EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt` + `EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt` |
| new_y8m_alone | 63.5 | 55.8 | `EXP-2026-10-28-02-best-recipe-2710/runs/new_y8m/weights/best.pt` |
| old_exp11_y8m_dfrozen | 63.6 | 55.0 | `EXP-2026-10-14-11-d-frozen-backbone/runs/ft/weights/best.pt` |
| old_exp12_y8mft_r | 63.2 | 54.6 | `EXP-2026-10-14-12-radar-r-stowed-vehicle/runs/ft/weights/best.pt` |
| old_y11l_alone | 64.4 | 54.6 | `models/BEST/prod_y11l_best.pt` |
| old_exp02_y8mft_c | 63.0 | 54.4 | `EXP-2026-10-14-02-relabel-c-unloaded-vehicle/runs/ft/weights/best.pt` |
| old_exp1501_ens_y11l_y11ld | 64.3 | 54.2 | `models/BEST/prod_y11l_best.pt` + `EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt` |
| old_exp04_y8mft_control | 62.9 | 54.2 | `EXP-2026-10-14-04-control-finetune-a-unchanged/runs/ft/weights/best.pt` |
| old_exp03_y8mft_d | 62.7 | 54.2 | `EXP-2026-10-14-03-relabel-d-both-vehicle/runs/ft/weights/best.pt` |
| old_exp01_y8mft_b | 62.9 | 54.1 | `EXP-2026-10-14-01-relabel-b-nonlauncher-vehicle/runs/ft/weights/best.pt` |
| old_y8m_alone | 62.3 | 54.1 | `models/BEST/prod_y8m_best.pt` |
| old_exp18_y11lft_control | 63.9 | 53.9 | `EXP-2026-10-14-18-y11l-control-finetune-a/runs/ft/weights/best.pt` |
| old_exp1502_y11lft_d_lowlr | 63.4 | 53.8 | `EXP-2026-10-15-02-y11l-finetune-d-lowlr/runs/ft/weights/best.pt` |
| old_exp19_y11lft_dr | 61.7 | 53.7 | `EXP-2026-10-14-19-y11l-finetune-dr/runs/ft/weights/best.pt` |
| old_exp13_y11l_ftD | 63.8 | 53.1 | `EXP-2026-10-14-13-y11l-finetune-d/runs/ft/weights/best.pt` |
