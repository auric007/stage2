# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6806 after thresholding -> **5705** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 246, tank 99, camo 81, smallvehicle 16

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 64 | 93 | 40.8 | 35.8 |
| camo | 750 | 264 | 330 | 44.4 | 35.2 |
| logistic | 241 | 60 | 138 | 30.3 | 24.9 |
| missile | 162 | 67 | 155 | 30.2 | 41.4 |
| radar | 176 | 92 | 132 | 41.1 | 52.3 |
| smallvehicle | 822 | 502 | 714 | 41.3 | 61.1 |
| tank | 729 | 475 | 229 | 67.5 | 65.2 |
| vehicle | 2041 | 1535 | 855 | 64.2 | 75.2 |
| **MICRO** | 5100 | 3059 | 2646 | 53.6 | 60.0 |
