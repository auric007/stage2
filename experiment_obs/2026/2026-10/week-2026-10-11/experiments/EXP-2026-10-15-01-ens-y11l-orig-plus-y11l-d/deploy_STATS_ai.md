# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6810 after thresholding -> **5751** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 279, camo 84, tank 80, smallvehicle 17

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 43 | 130 | 24.9 | 24.0 |
| camo | 750 | 252 | 288 | 46.7 | 33.6 |
| logistic | 241 | 74 | 202 | 26.8 | 30.7 |
| missile | 162 | 63 | 138 | 31.3 | 38.9 |
| radar | 176 | 92 | 140 | 39.7 | 52.3 |
| smallvehicle | 822 | 481 | 639 | 42.9 | 58.5 |
| tank | 729 | 471 | 264 | 64.1 | 64.6 |
| vehicle | 2041 | 1552 | 922 | 62.7 | 76.0 |
| **MICRO** | 5100 | 3028 | 2723 | 52.7 | 59.4 |
