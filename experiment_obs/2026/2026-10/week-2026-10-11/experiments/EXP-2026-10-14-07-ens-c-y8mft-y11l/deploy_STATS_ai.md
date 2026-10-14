# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6810 after thresholding -> **5636** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 280, tank 123, camo 94, smallvehicle 18

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 65 | 127 | 33.9 | 36.3 |
| camo | 750 | 262 | 320 | 45.0 | 34.9 |
| logistic | 241 | 57 | 131 | 30.3 | 23.7 |
| missile | 162 | 70 | 185 | 27.5 | 43.2 |
| radar | 176 | 91 | 151 | 37.6 | 51.7 |
| smallvehicle | 822 | 470 | 653 | 41.9 | 57.2 |
| tank | 729 | 449 | 206 | 68.5 | 61.6 |
| vehicle | 2041 | 1530 | 869 | 63.8 | 75.0 |
| **MICRO** | 5100 | 2994 | 2642 | 53.1 | 58.7 |
