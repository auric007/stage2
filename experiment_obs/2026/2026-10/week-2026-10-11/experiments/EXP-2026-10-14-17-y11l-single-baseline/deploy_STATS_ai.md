# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4926 after thresholding -> **4520** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 65, camo 39, tank 25, smallvehicle 2

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 28 | 46 | 37.8 | 15.6 |
| camo | 750 | 240 | 198 | 54.8 | 32.0 |
| logistic | 241 | 39 | 67 | 36.8 | 16.2 |
| missile | 162 | 53 | 108 | 32.9 | 32.7 |
| radar | 176 | 85 | 94 | 47.5 | 48.3 |
| smallvehicle | 822 | 450 | 414 | 52.1 | 54.7 |
| tank | 729 | 471 | 247 | 65.6 | 64.6 |
| vehicle | 2041 | 1414 | 566 | 71.4 | 69.3 |
| **MICRO** | 5100 | 2780 | 1740 | 61.5 | 54.5 |
