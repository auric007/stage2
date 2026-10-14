# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 5575 after thresholding -> **5087** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 92, camo 46, tank 23, smallvehicle 8

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 50 | 57 | 46.7 | 27.9 |
| camo | 750 | 266 | 267 | 49.9 | 35.5 |
| logistic | 241 | 43 | 112 | 27.7 | 17.8 |
| missile | 162 | 39 | 78 | 33.3 | 24.1 |
| radar | 176 | 78 | 125 | 38.4 | 44.3 |
| smallvehicle | 822 | 476 | 637 | 42.8 | 57.9 |
| tank | 729 | 415 | 167 | 71.3 | 56.9 |
| vehicle | 2041 | 1504 | 773 | 66.1 | 73.7 |
| **MICRO** | 5100 | 2871 | 2216 | 56.4 | 56.3 |
