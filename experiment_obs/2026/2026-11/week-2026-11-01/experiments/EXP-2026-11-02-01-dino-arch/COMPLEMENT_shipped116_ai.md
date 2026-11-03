# Which instances does each model find? 116 scenes, IoU 0.3, official labels_simple

H = hybrid (each class from its own group, before the merge, deploy threshold). D = shipped DINO, class-aware, at its best-F1 threshold per class (arty 0.25, camo 0.7, logistic 0.525, missile 0.325, radar 0.5, smallvehicle 0.4, tank 0.4, vehicle 0.3).

## 1. Same class, different instances: hybrid vs DINO

| class | GT | H recall | D recall | both | only H | only D | neither | union recall | overlap (Jaccard of found sets) |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| arty | 249 | 60.6 | 21.7 | 43 | 108 | 11 | 87 | **65.1** | 0.27 |
| camo | 558 | 64.0 | 53.9 | 250 | 107 | 51 | 150 | **73.1** | 0.61 |
| logistic | 298 | 33.2 | 27.2 | 53 | 46 | 28 | 171 | **42.6** | 0.42 |
| missile | 413 | 59.8 | 38.3 | 140 | 107 | 18 | 148 | **64.2** | 0.53 |
| radar | 275 | 67.3 | 51.3 | 122 | 63 | 19 | 71 | **74.2** | 0.60 |
| smallvehicle | 1242 | 63.0 | 63.3 | 597 | 185 | 189 | 271 | **78.2** | 0.61 |
| tank | 645 | 83.7 | 65.4 | 407 | 133 | 15 | 90 | **86.0** | 0.73 |
| vehicle | 3098 | 84.6 | 47.1 | 1402 | 1220 | 56 | 420 | **86.4** | 0.52 |

## 2. Found, but given the wrong class (localization vs classification)

Any box = any class at the deploy thresholds (H) / objectness >= 0.3 (D). 'Wrong class' = a box is there but not of the right class.

| class | GT | H: any box | H: right class | H: wrong class → said as | D: any box | D: right class | D: wrong class → said as |
|---|--:|--:|--:|---|--:|--:|---|
| arty | 249 | 73.1 | 60.6 | tank 16, vehicle 7, camo 6 | 86.7 | 21.7 | tank 70, vehicle 40, camo 34 |
| camo | 558 | 70.1 | 64.0 | vehicle 17, tank 6, radar 3 | 85.5 | 53.9 | tank 17, vehicle 8, radar 5 |
| logistic | 298 | 65.8 | 33.2 | vehicle 85, tank 6, smallvehicle 2 | 77.2 | 27.2 | vehicle 44, camo 25, smallvehicle 9 |
| missile | 413 | 92.5 | 59.8 | vehicle 104, radar 13, tank 6 | 98.1 | 38.3 | camo 122, vehicle 87, radar 20 |
| radar | 275 | 85.8 | 67.3 | vehicle 36, tank 4, arty 4 | 92.4 | 51.3 | camo 33, vehicle 30, missile 19 |
| smallvehicle | 1242 | 70.8 | 63.0 | vehicle 74, logistic 16, missile 1 | 87.8 | 63.3 | logistic 168, vehicle 32, tank 6 |
| tank | 645 | 94.9 | 83.7 | vehicle 46, camo 23, arty 3 | 96.3 | 65.4 | camo 141, vehicle 32, logistic 6 |
| vehicle | 3098 | 88.4 | 84.6 | smallvehicle 57, logistic 14, tank 12 | 92.5 | 47.1 | logistic 821, camo 219, missile 142 |

## 3a. missile: two YOLO sources the researcher compared (exp15 vs exp09) + DINO

exp15: 247/413 (59.8) · exp09: 254/413 (61.5) · both 246 · only exp15 1 · only exp09 8 · union 255 (61.7) · Jaccard 0.96. All three incl. D: 66.1.

| child class | GT | exp15 recall | exp09 recall | D recall |
|---|--:|--:|--:|--:|
| HQ9 | 89 | 73.0 | 75.3 | 37.1 |
| MLRS | 82 | 24.4 | 26.8 | 6.1 |
| S300 | 62 | 80.6 | 80.6 | 80.6 |
| MissileSystem | 55 | 56.4 | 56.4 | 32.7 |
| HQ12 | 37 | 62.2 | 64.9 | 56.8 |
| HQ22 | 33 | 84.8 | 84.8 | 60.6 |
| HQ16 | 24 | 66.7 | 75.0 | 25.0 |
| HQ2 | 14 | 92.9 | 92.9 | 14.3 |
| HQ6 | 10 | 10.0 | 10.0 | 20.0 |
| SPADA | 4 | 0.0 | 0.0 | 0.0 |
| HQ6A | 3 | 0.0 | 0.0 | 33.3 |

| SOC (what the label is based on) | GT | exp15 recall | exp09 recall | D recall |
|---|--:|--:|--:|--:|
| Pixels | 249 | 71.5 | 73.1 | 49.4 |
| Nil | 82 | 24.4 | 26.8 | 6.1 |
| Third_Party | 63 | 63.5 | 65.1 | 36.5 |
| Site | 19 | 47.4 | 47.4 | 36.8 |

## 3b. radar: two YOLO sources the researcher compared (ft6 vs new_ens_last) + DINO

ft6: 185/275 (67.3) · new_ens_last: 175/275 (63.6) · both 167 · only ft6 18 · only new_ens_last 8 · union 193 (70.2) · Jaccard 0.87. All three incl. D: 76.4.

| child class | GT | ft6 recall | new_ens_last recall | D recall |
|---|--:|--:|--:|--:|
| RadarSystem | 126 | 57.1 | 56.3 | 46.8 |
| EWS | 28 | 21.4 | 21.4 | 7.1 |
| HT233 | 19 | 94.7 | 84.2 | 52.6 |
| Type780 | 14 | 85.7 | 71.4 | 28.6 |
| H200 | 12 | 66.7 | 66.7 | 75.0 |
| 96L6ECheeseBoard | 10 | 100.0 | 90.0 | 50.0 |
| FlapLidMVC | 10 | 100.0 | 90.0 | 70.0 |
| JY27A | 9 | 88.9 | 88.9 | 88.9 |
| YLC2V | 6 | 83.3 | 100.0 | 83.3 |
| LX15 | 4 | 100.0 | 100.0 | 100.0 |
| 64N6BigBird | 4 | 100.0 | 100.0 | 100.0 |
| YLC4 | 4 | 100.0 | 100.0 | 100.0 |
| JLG43D | 3 | 100.0 | 100.0 | 100.0 |
| YLC2E | 3 | 100.0 | 100.0 | 100.0 |
| REL3 | 3 | 100.0 | 66.7 | 100.0 |
| BarLock | 3 | 66.7 | 66.7 | 33.3 |
| YLC6 | 2 | 50.0 | 50.0 | 0.0 |
| YLC8B | 2 | 0.0 | 0.0 | 0.0 |
| TG680 | 2 | 100.0 | 100.0 | 100.0 |
| YLC18 | 1 | 100.0 | 100.0 | 100.0 |
| JY9WallEye | 1 | 100.0 | 100.0 | 100.0 |
| 702D | 1 | 0.0 | 0.0 | 0.0 |
| REL1 | 1 | 100.0 | 100.0 | 100.0 |
| LX10 | 1 | 100.0 | 100.0 | 100.0 |
| Type305A | 1 | 100.0 | 0.0 | 100.0 |
| JY11B | 1 | 100.0 | 100.0 | 100.0 |
| REL4 | 1 | 100.0 | 100.0 | 100.0 |
| JY27WideMat | 1 | 100.0 | 0.0 | 0.0 |
| JY11 | 1 | 100.0 | 0.0 | 0.0 |
| Type120 | 1 | 100.0 | 100.0 | 100.0 |

| SOC (what the label is based on) | GT | ft6 recall | new_ens_last recall | D recall |
|---|--:|--:|--:|--:|
| Pixels | 205 | 69.3 | 63.9 | 54.1 |
| Third_Party | 40 | 52.5 | 55.0 | 45.0 |
| Site | 30 | 73.3 | 73.3 | 40.0 |
