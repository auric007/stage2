# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4954 after thresholding -> **4561** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 50, tank 34, camo 30, smallvehicle 1

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 47 | 44 | 51.6 | 26.3 |
| camo | 750 | 276 | 305 | 47.5 | 36.8 |
| logistic | 241 | 26 | 48 | 35.1 | 10.8 |
| missile | 162 | 57 | 99 | 36.5 | 35.2 |
| radar | 176 | 73 | 77 | 48.7 | 41.5 |
| smallvehicle | 822 | 463 | 542 | 46.1 | 56.3 |
| tank | 729 | 468 | 189 | 71.2 | 64.2 |
| vehicle | 2041 | 1348 | 499 | 73.0 | 66.0 |
| **MICRO** | 5100 | 2758 | 1803 | 60.5 | 54.1 |
