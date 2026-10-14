# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4877 after thresholding -> **4465** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 59, tank 46, camo 35, smallvehicle 3

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 48 | 63 | 43.2 | 26.8 |
| camo | 750 | 279 | 272 | 50.6 | 37.2 |
| logistic | 241 | 17 | 34 | 33.3 | 7.1 |
| missile | 162 | 53 | 114 | 31.7 | 32.7 |
| radar | 176 | 77 | 82 | 48.4 | 43.8 |
| smallvehicle | 822 | 417 | 485 | 46.2 | 50.7 |
| tank | 729 | 437 | 189 | 69.8 | 59.9 |
| vehicle | 2041 | 1332 | 566 | 70.2 | 65.3 |
| **MICRO** | 5100 | 2660 | 1805 | 59.6 | 52.2 |
