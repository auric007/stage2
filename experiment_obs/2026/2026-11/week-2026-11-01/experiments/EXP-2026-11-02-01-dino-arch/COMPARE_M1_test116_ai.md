# DINO checkpoints on Test116 vs HYBRID (IoU 0.3, official labels_simple, Delete-centred preds dropped, 0.30 m grid)

Only two M1 checkpoints exist: epoch 1 (best class-aware val, 54.6) and epoch 13 (last, val 53.1); epochs 2-12 were not kept.

## 1. Overall

| model / route | P | R | MICRO | MACRO | TP / FP / FN |
|---|--:|--:|--:|--:|---|
| **HYBRID deployed** | 61.8 | 66.1 | 63.9 | 56.7 | 4482 / 2767 / 2296 |
| DINO shipped (29-Oct): DINO alone (best-F1 thr/class) | 59.4 | 50.2 | 54.4 | 48.9 | 3401 / 2326 / 3377 |
| M1 epoch 1 (val best): DINO alone (best-F1 thr/class) | 62.0 | 54.6 | 58.1 | 51.6 | 3702 / 2273 / 3076 |
| M1 epoch 13 (last): DINO alone (best-F1 thr/class) | 65.8 | 61.6 | 63.7 | 53.7 | 4176 / 2167 / 2602 |
| DINO shipped (29-Oct): ENS best-F1 | 65.4 | 67.5 | 66.4 | 59.5 | 4574 / 2424 / 2204 |
| M1 epoch 1 (val best): ENS best-F1 | 65.1 | 66.9 | 66.0 | 59.2 | 4532 / 2428 / 2246 |
| M1 epoch 13 (last): ENS best-F1 | 65.3 | 66.7 | 66.0 | 59.4 | 4518 / 2402 / 2260 |
| DINO shipped (29-Oct): ENS recall-leaning | 57.9 | 70.1 | 63.4 | 56.3 | 4748 / 3454 / 2030 |
| M1 epoch 1 (val best): ENS recall-leaning | 58.1 | 68.8 | 63.0 | 56.3 | 4665 / 3369 / 2113 |
| M1 epoch 13 (last): ENS recall-leaning | 58.4 | 69.7 | 63.5 | 56.9 | 4722 / 3366 / 2056 |

## 2. Per class F1 (TP/FP/FN)

| class | HYBRID | DINO shipped (29-Oct): DINO alone | M1 epoch 1 (val best): DINO alone | M1 epoch 13 (last): DINO alone | DINO shipped (29-Oct): ENS recall | M1 epoch 1 (val best): ENS recall | M1 epoch 13 (last): ENS recall |
|---|---|---|---|---|---|---|---|
| arty | 49.2 (137/171/112) | 32.8 (54/26/195) | 32.4 (50/10/199) | 33.3 (61/56/188) | 51.9 (135/136/114) | 51.0 (146/177/103) | 53.1 (138/133/111) |
| camo | 56.6 (315/240/243) | 54.9 (301/237/257) | 53.2 (301/272/257) | 54.4 (300/244/258) | 55.3 (343/339/215) | 54.9 (333/323/225) | 54.3 (328/323/230) |
| logistic | 31.3 (99/235/199) | 15.1 (81/695/217) | 21.6 (83/389/215) | 17.8 (54/256/244) | 31.3 (99/235/199) | 31.7 (102/244/196) | 31.3 (99/235/199) |
| missile | 52.7 (221/205/192) | 42.1 (158/179/255) | 52.3 (189/121/224) | 56.2 (198/93/215) | 52.3 (225/223/188) | 52.9 (227/218/186) | 53.7 (233/221/180) |
| radar | 57.5 (185/183/90) | 56.1 (141/87/134) | 59.1 (140/59/135) | 58.8 (133/44/142) | 58.2 (191/190/84) | 57.7 (186/184/89) | 58.2 (190/188/85) |
| smallvehicle | 62.1 (775/479/467) | 62.7 (786/479/456) | 61.4 (829/629/413) | 63.8 (783/431/459) | 65.1 (870/560/372) | 65.1 (868/558/374) | 64.8 (864/562/378) |
| tank | 73.4 (447/126/198) | 69.1 (422/155/223) | 71.2 (446/161/199) | 75.0 (436/81/209) | 67.4 (486/312/159) | 68.7 (471/256/174) | 71.7 (490/232/155) |
| vehicle | 70.5 (2303/1128/795) | 58.0 (1458/468/1640) | 61.7 (1664/632/1434) | 70.5 (2211/962/887) | 69.0 (2399/1459/699) | 68.2 (2332/1409/766) | 68.5 (2380/1472/718) |

## 3. Recall-leaning ENS, radar / missile, by merge rule (missile base = exp15)

| model | rule | radar P / R · TP/FP/FN | missile P / R · TP/FP/FN | all P / R | MICRO | MACRO |
|---|---|---|---|---|--:|--:|
| DINO shipped (29-Oct) | P (current) | 50.1 / 69.5 · 191/190/84 | 50.2 / 54.5 · 225/223/188 | 57.9 / 70.1 | 63.4 | 56.3 |
| DINO shipped (29-Oct) | C0.25 | 53.4 / 66.2 · 182/159/93 | 50.6 / 59.8 · 247/241/166 | 58.0 / 70.3 | 63.6 | 56.7 |
| DINO shipped (29-Oct) | B (radar+missile both kept) | 50.1 / 69.5 · 191/190/84 | 49.4 / 60.8 · 251/257/162 | 57.8 / 70.4 | 63.5 | 56.6 |
| M1 epoch 1 (val best) | P (current) | 50.3 / 67.6 · 186/184/89 | 51.0 / 55.0 · 227/218/186 | 58.1 / 68.8 | 63.0 | 56.3 |
| M1 epoch 1 (val best) | C0.25 | 53.6 / 64.4 · 177/153/98 | 51.3 / 60.3 · 249/236/164 | 58.2 / 69.0 | 63.2 | 56.7 |
| M1 epoch 1 (val best) | B (radar+missile both kept) | 50.3 / 67.6 · 186/184/89 | 50.1 / 61.3 · 253/252/160 | 58.0 / 69.2 | 63.1 | 56.5 |
| M1 epoch 13 (last) | P (current) | 50.3 / 69.1 · 190/188/85 | 51.3 / 56.4 · 233/221/180 | 58.4 / 69.7 | 63.5 | 56.9 |
| M1 epoch 13 (last) | C0.25 | 53.6 / 65.8 · 181/157/94 | 51.6 / 61.7 · 255/239/158 | 58.5 / 69.9 | 63.7 | 57.4 |
| M1 epoch 13 (last) | B (radar+missile both kept) | 50.3 / 69.1 · 190/188/85 | 50.4 / 62.7 · 259/255/154 | 58.3 / 70.1 | 63.6 | 57.2 |

## 4. Recall-leaning picks per checkpoint (recall_picks.py)

- DINO shipped (29-Oct): radar M3@0.8, missile M3@0.325, camo M3@0.7, arty M2@0.175, smallvehicle M3@0.55, tank M2@0.1, vehicle M2@0.275, logistic H@0
- M1 epoch 1 (val best): radar M3@0.8, missile M3@0.55, camo M2@0.55, arty M3@0.525, smallvehicle M3@0.525, tank M2@0.3, vehicle M2@0.4, logistic M3@0.6
- M1 epoch 13 (last): radar M3@0.85, missile M3@0.4, camo M3@0.75, arty M2@0.275, smallvehicle M3@0.6, tank M2@0.175, vehicle M2@0.475, logistic H@0
