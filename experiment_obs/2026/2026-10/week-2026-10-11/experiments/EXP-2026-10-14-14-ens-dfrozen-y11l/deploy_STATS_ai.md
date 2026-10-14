# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 7207 after thresholding -> **5971** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 301, camo 102, tank 99, smallvehicle 22

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 64 | 105 | 37.9 | 35.8 |
| camo | 750 | 263 | 311 | 45.8 | 35.1 |
| logistic | 241 | 69 | 215 | 24.3 | 28.6 |
| missile | 162 | 63 | 134 | 32.0 | 38.9 |
| radar | 176 | 92 | 174 | 34.6 | 52.3 |
| smallvehicle | 822 | 505 | 780 | 39.3 | 61.4 |
| tank | 729 | 433 | 211 | 67.2 | 59.4 |
| vehicle | 2041 | 1572 | 980 | 61.6 | 77.0 |
| **MICRO** | 5100 | 3061 | 2910 | 51.3 | 60.0 |
