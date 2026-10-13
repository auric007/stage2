# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4983 after thresholding -> **4553** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 71, tank 53, camo 41, smallvehicle 3

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 49 | 85 | 36.6 | 27.4 |
| camo | 750 | 271 | 282 | 49.0 | 36.1 |
| logistic | 241 | 18 | 36 | 33.3 | 7.5 |
| missile | 162 | 64 | 140 | 31.4 | 39.5 |
| radar | 176 | 76 | 92 | 45.2 | 43.2 |
| smallvehicle | 822 | 414 | 493 | 45.6 | 50.4 |
| tank | 729 | 436 | 200 | 68.6 | 59.8 |
| vehicle | 2041 | 1325 | 572 | 69.8 | 64.9 |
| **MICRO** | 5100 | 2653 | 1900 | 58.3 | 52.0 |
