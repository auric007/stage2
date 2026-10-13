# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4939 after thresholding -> **4498** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 63, tank 48, camo 38, smallvehicle 2

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 49 | 67 | 42.2 | 27.4 |
| camo | 750 | 283 | 267 | 51.5 | 37.7 |
| logistic | 241 | 19 | 36 | 34.5 | 7.9 |
| missile | 162 | 59 | 139 | 29.8 | 36.4 |
| radar | 176 | 77 | 77 | 50.0 | 43.8 |
| smallvehicle | 822 | 417 | 484 | 46.3 | 50.7 |
| tank | 729 | 444 | 188 | 70.3 | 60.9 |
| vehicle | 2041 | 1320 | 572 | 69.8 | 64.7 |
| **MICRO** | 5100 | 2668 | 1830 | 59.3 | 52.3 |
