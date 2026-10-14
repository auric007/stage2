# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6778 after thresholding -> **5635** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 260, tank 107, camo 92, smallvehicle 16

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 63 | 114 | 35.6 | 35.2 |
| camo | 750 | 266 | 308 | 46.3 | 35.5 |
| logistic | 241 | 60 | 136 | 30.6 | 24.9 |
| missile | 162 | 68 | 175 | 28.0 | 42.0 |
| radar | 176 | 91 | 138 | 39.7 | 51.7 |
| smallvehicle | 822 | 477 | 665 | 41.8 | 58.0 |
| tank | 729 | 454 | 205 | 68.9 | 62.3 |
| vehicle | 2041 | 1528 | 887 | 63.3 | 74.9 |
| **MICRO** | 5100 | 3007 | 2628 | 53.4 | 59.0 |
