# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4929 after thresholding -> **4485** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: tank 71, vehicle 64, camo 38, smallvehicle 3

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 47 | 85 | 35.6 | 26.3 |
| camo | 750 | 280 | 265 | 51.4 | 37.3 |
| logistic | 241 | 17 | 38 | 30.9 | 7.1 |
| missile | 162 | 66 | 172 | 27.7 | 40.7 |
| radar | 176 | 74 | 75 | 49.7 | 42.0 |
| smallvehicle | 822 | 410 | 478 | 46.2 | 49.9 |
| tank | 729 | 442 | 204 | 68.4 | 60.6 |
| vehicle | 2041 | 1302 | 530 | 71.1 | 63.8 |
| **MICRO** | 5100 | 2638 | 1847 | 58.8 | 51.7 |
