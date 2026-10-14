# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6798 after thresholding -> **5647** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 282, tank 129, camo 89, smallvehicle 14

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 66 | 138 | 32.4 | 36.9 |
| camo | 750 | 266 | 316 | 45.7 | 35.5 |
| logistic | 241 | 56 | 139 | 28.7 | 23.2 |
| missile | 162 | 73 | 201 | 26.6 | 45.1 |
| radar | 176 | 84 | 127 | 39.8 | 47.7 |
| smallvehicle | 822 | 475 | 670 | 41.5 | 57.8 |
| tank | 729 | 451 | 203 | 69.0 | 61.9 |
| vehicle | 2041 | 1522 | 860 | 63.9 | 74.6 |
| **MICRO** | 5100 | 2993 | 2654 | 53.0 | 58.7 |
