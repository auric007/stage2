# Which instances does each model find? 116 scenes, IoU 0.3, official labels_simple

H = hybrid (each class from its own group, before the merge, deploy threshold). D = shipped DINO, class-aware, at its best-F1 threshold per class (arty 0.3, camo 0.625, logistic 0.4, missile 0.45, radar 0.75, smallvehicle 0.475, tank 0.3, vehicle 0.575).

## 1. Same class, different instances: hybrid vs DINO

| class | GT | H recall | D recall | both | only H | only D | neither | union recall | overlap (Jaccard of found sets) |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| arty | 249 | 60.6 | 24.5 | 43 | 108 | 18 | 80 | **67.9** | 0.25 |
| camo | 558 | 64.0 | 53.8 | 266 | 91 | 34 | 167 | **70.1** | 0.68 |
| logistic | 298 | 33.2 | 18.1 | 35 | 64 | 19 | 180 | **39.6** | 0.30 |
| missile | 413 | 59.8 | 47.9 | 174 | 73 | 24 | 142 | **65.6** | 0.64 |
| radar | 275 | 67.3 | 48.4 | 125 | 60 | 8 | 82 | **70.2** | 0.65 |
| smallvehicle | 1242 | 63.0 | 63.0 | 615 | 167 | 168 | 292 | **76.5** | 0.65 |
| tank | 645 | 83.7 | 67.6 | 417 | 123 | 19 | 86 | **86.7** | 0.75 |
| vehicle | 3098 | 84.6 | 71.4 | 2107 | 515 | 104 | 372 | **88.0** | 0.77 |

## 2. Found, but given the wrong class (localization vs classification)

Any box = any class at the deploy thresholds (H) / objectness >= 0.3 (D). 'Wrong class' = a box is there but not of the right class.

| class | GT | H: any box | H: right class | H: wrong class → said as | D: any box | D: right class | D: wrong class → said as |
|---|--:|--:|--:|---|--:|--:|---|
| arty | 249 | 73.1 | 60.6 | tank 16, vehicle 7, camo 6 | 85.5 | 24.5 | vehicle 107, radar 19, tank 13 |
| camo | 558 | 70.1 | 64.0 | vehicle 17, tank 6, radar 3 | 81.9 | 53.8 | vehicle 37, tank 22, missile 14 |
| logistic | 298 | 65.8 | 33.2 | vehicle 85, tank 6, smallvehicle 2 | 76.5 | 18.1 | vehicle 147, smallvehicle 7, camo 6 |
| missile | 413 | 92.5 | 59.8 | vehicle 104, radar 13, tank 6 | 97.8 | 47.9 | vehicle 176, radar 14, logistic 5 |
| radar | 275 | 85.8 | 67.3 | vehicle 36, tank 4, arty 4 | 91.6 | 48.4 | vehicle 73, missile 11, camo 4 |
| smallvehicle | 1242 | 70.8 | 63.0 | vehicle 74, logistic 16, missile 1 | 88.2 | 63.0 | vehicle 148, logistic 20, radar 3 |
| tank | 645 | 94.9 | 83.7 | vehicle 46, camo 23, arty 3 | 97.5 | 67.6 | vehicle 144, camo 39, logistic 7 |
| vehicle | 3098 | 88.4 | 84.6 | smallvehicle 57, logistic 14, tank 12 | 92.4 | 71.4 | smallvehicle 125, logistic 73, missile 62 |

## 3a. missile: two YOLO sources the researcher compared (exp15 vs exp09) + DINO

exp15: 247/413 (59.8) · exp09: 254/413 (61.5) · both 246 · only exp15 1 · only exp09 8 · union 255 (61.7) · Jaccard 0.96. All three incl. D: 67.3.

| child class | GT | exp15 recall | exp09 recall | D recall |
|---|--:|--:|--:|--:|
| HQ9 | 89 | 73.0 | 75.3 | 66.3 |
| MLRS | 82 | 24.4 | 26.8 | 1.2 |
| S300 | 62 | 80.6 | 80.6 | 80.6 |
| MissileSystem | 55 | 56.4 | 56.4 | 56.4 |
| HQ12 | 37 | 62.2 | 64.9 | 67.6 |
| HQ22 | 33 | 84.8 | 84.8 | 69.7 |
| HQ16 | 24 | 66.7 | 75.0 | 8.3 |
| HQ2 | 14 | 92.9 | 92.9 | 14.3 |
| HQ6 | 10 | 10.0 | 10.0 | 40.0 |
| SPADA | 4 | 0.0 | 0.0 | 0.0 |
| HQ6A | 3 | 0.0 | 0.0 | 33.3 |

| SOC (what the label is based on) | GT | exp15 recall | exp09 recall | D recall |
|---|--:|--:|--:|--:|
| Pixels | 249 | 71.5 | 73.1 | 61.0 |
| Nil | 82 | 24.4 | 26.8 | 1.2 |
| Third_Party | 63 | 63.5 | 65.1 | 54.0 |
| Site | 19 | 47.4 | 47.4 | 57.9 |

## 3b. radar: two YOLO sources the researcher compared (ft6 vs new_ens_last) + DINO

ft6: 185/275 (67.3) · new_ens_last: 175/275 (63.6) · both 167 · only ft6 18 · only new_ens_last 8 · union 193 (70.2) · Jaccard 0.87. All three incl. D: 72.7.

| child class | GT | ft6 recall | new_ens_last recall | D recall |
|---|--:|--:|--:|--:|
| RadarSystem | 126 | 57.1 | 56.3 | 40.5 |
| EWS | 28 | 21.4 | 21.4 | 3.6 |
| HT233 | 19 | 94.7 | 84.2 | 78.9 |
| Type780 | 14 | 85.7 | 71.4 | 35.7 |
| H200 | 12 | 66.7 | 66.7 | 58.3 |
| 96L6ECheeseBoard | 10 | 100.0 | 90.0 | 40.0 |
| FlapLidMVC | 10 | 100.0 | 90.0 | 50.0 |
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
| Pixels | 205 | 69.3 | 63.9 | 49.8 |
| Third_Party | 40 | 52.5 | 55.0 | 42.5 |
| Site | 30 | 73.3 | 73.3 | 46.7 |
