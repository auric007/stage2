# Post-processed predictions — priority=['radar', 'missile', 'arty', 'logistic'], dedup IoU=0.4

Thresholds: `arty=0.86`, `camo=0.42`, `logistic=0.77`, `missile=0.85`, `radar=0.39`, `smallvehicle=0.24`, `tank=0.54`, `vehicle=0.84`

Boxes: 6770 after thresholding -> **5605** after cross-class de-dup (one box per object).

Dropped because a PRIORITY box owned the region: vehicle 289, tank 127, camo 89, smallvehicle 15

| class | GT | TP | FP | precision % | recall % |
|---|--:|--:|--:|--:|--:|
| arty | 179 | 63 | 133 | 32.1 | 35.2 |
| camo | 750 | 270 | 307 | 46.8 | 36.0 |
| logistic | 241 | 57 | 136 | 29.5 | 23.7 |
| missile | 162 | 72 | 204 | 26.1 | 44.4 |
| radar | 176 | 90 | 138 | 39.5 | 51.1 |
| smallvehicle | 822 | 476 | 647 | 42.4 | 57.9 |
| tank | 729 | 456 | 202 | 69.3 | 62.6 |
| vehicle | 2041 | 1513 | 841 | 64.3 | 74.1 |
| **MICRO** | 5100 | 2997 | 2608 | 53.5 | 58.8 |
