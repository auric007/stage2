# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 5162 after thresholding -> **4702** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 67, camo 44, tank 32, smallvehicle 4

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 30 | 46 | 39.5 | 16.8 |
| camo | 750 | 253 | 276 | 47.8 | 33.7 |
| logistic | 241 | 42 | 49 | 46.2 | 17.4 |
| missile | 162 | 44 | 80 | 35.5 | 27.2 |
| radar | 176 | 85 | 93 | 47.8 | 48.3 |
| smallvehicle | 822 | 442 | 478 | 48.0 | 53.8 |
| tank | 729 | 496 | 303 | 62.1 | 68.0 |
| vehicle | 2041 | 1412 | 573 | 71.1 | 69.2 |
| **MICRO** | 5100 | 2804 | 1898 | 59.6 | 55.0 |
