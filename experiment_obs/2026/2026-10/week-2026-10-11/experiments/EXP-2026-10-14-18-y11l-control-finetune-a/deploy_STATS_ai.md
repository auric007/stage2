# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 5034 after thresholding -> **4594** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 96, camo 35, tank 18, smallvehicle 3

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 32 | 43 | 42.7 | 17.9 |
| camo | 750 | 252 | 228 | 52.5 | 33.6 |
| logistic | 241 | 41 | 87 | 32.0 | 17.0 |
| missile | 162 | 60 | 129 | 31.7 | 37.0 |
| radar | 176 | 78 | 107 | 42.2 | 44.3 |
| smallvehicle | 822 | 460 | 441 | 51.1 | 56.0 |
| tank | 729 | 475 | 259 | 64.7 | 65.2 |
| vehicle | 2041 | 1378 | 524 | 72.5 | 67.5 |
| **MICRO** | 5100 | 2776 | 1818 | 60.4 | 54.4 |
