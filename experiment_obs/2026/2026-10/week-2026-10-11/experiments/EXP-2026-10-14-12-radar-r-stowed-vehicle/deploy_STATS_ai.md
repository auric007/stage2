# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 4984 after thresholding -> **4560** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 65, tank 62, camo 35, smallvehicle 2

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 54 | 88 | 38.0 | 30.2 |
| camo | 750 | 276 | 269 | 50.6 | 36.8 |
| logistic | 241 | 19 | 37 | 33.9 | 7.9 |
| missile | 162 | 68 | 168 | 28.8 | 42.0 |
| radar | 176 | 70 | 60 | 53.8 | 39.8 |
| smallvehicle | 822 | 427 | 507 | 45.7 | 51.9 |
| tank | 729 | 439 | 191 | 69.7 | 60.2 |
| vehicle | 2041 | 1336 | 551 | 70.8 | 65.5 |
| **MICRO** | 5100 | 2689 | 1871 | 59.0 | 52.7 |
