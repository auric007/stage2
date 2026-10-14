# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4642 after thresholding -> **4221** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 47, camo 44, tank 25, smallvehicle 2

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 26 | 45 | 36.6 | 14.5 |
| camo | 750 | 297 | 393 | 43.0 | 39.6 |
| logistic | 241 | 41 | 38 | 51.9 | 17.0 |
| missile | 162 | 39 | 65 | 37.5 | 24.1 |
| radar | 176 | 77 | 78 | 49.7 | 43.8 |
| smallvehicle | 822 | 400 | 283 | 58.6 | 48.7 |
| tank | 729 | 477 | 237 | 66.8 | 65.4 |
| vehicle | 2041 | 1237 | 488 | 71.7 | 60.6 |
| **MICRO** | 5100 | 2594 | 1627 | 61.5 | 50.9 |
