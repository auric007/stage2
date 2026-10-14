# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 7046 after thresholding -> **5859** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 297, tank 98, camo 90, smallvehicle 25

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 66 | 132 | 33.3 | 36.9 |
| camo | 750 | 272 | 335 | 44.8 | 36.3 |
| logistic | 241 | 63 | 156 | 28.8 | 26.1 |
| missile | 162 | 66 | 149 | 30.7 | 40.7 |
| radar | 176 | 90 | 152 | 37.2 | 51.1 |
| smallvehicle | 822 | 475 | 728 | 39.5 | 57.8 |
| tank | 729 | 464 | 220 | 67.8 | 63.6 |
| vehicle | 2041 | 1555 | 936 | 62.4 | 76.2 |
| **MICRO** | 5100 | 3051 | 2808 | 52.1 | 59.8 |
