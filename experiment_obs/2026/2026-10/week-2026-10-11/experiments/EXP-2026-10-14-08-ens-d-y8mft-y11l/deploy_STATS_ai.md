# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6758 after thresholding -> **5621** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 261, tank 104, camo 88, smallvehicle 18

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 64 | 108 | 37.2 | 35.8 |
| camo | 750 | 266 | 309 | 46.3 | 35.5 |
| logistic | 241 | 57 | 134 | 29.8 | 23.7 |
| missile | 162 | 68 | 165 | 29.2 | 42.0 |
| radar | 176 | 91 | 141 | 39.2 | 51.7 |
| smallvehicle | 822 | 475 | 663 | 41.7 | 57.8 |
| tank | 729 | 456 | 199 | 69.6 | 62.6 |
| vehicle | 2041 | 1541 | 884 | 63.5 | 75.5 |
| **MICRO** | 5100 | 3018 | 2603 | 53.7 | 59.2 |
