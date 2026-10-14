# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 5182 after thresholding -> **4738** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 94, camo 42, tank 17, smallvehicle 3

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 32 | 61 | 34.4 | 17.9 |
| camo | 750 | 249 | 241 | 50.8 | 33.2 |
| logistic | 241 | 48 | 106 | 31.2 | 19.9 |
| missile | 162 | 52 | 88 | 37.1 | 32.1 |
| radar | 176 | 81 | 102 | 44.3 | 46.0 |
| smallvehicle | 822 | 455 | 489 | 48.2 | 55.4 |
| tank | 729 | 468 | 256 | 64.6 | 64.2 |
| vehicle | 2041 | 1405 | 605 | 69.9 | 68.8 |
| **MICRO** | 5100 | 2790 | 1948 | 58.9 | 54.7 |
