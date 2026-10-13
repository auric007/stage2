# Deployment metrics on labels_detailed with camo credit — 80 test images

Model: sep_w1 FINAL ensemble at the deployment thresholds (`sep_w1/predictions_deploy`). IoU 0.3. Built 2026-10-09.

## Rule (agreed 2026-10-09)
- **GT class comes from `labels_detailed`** (true class, not the simple camo merge). `spa` → arty. aerial/radome/concealment dropped.
- **Camouflaged objects count as correct if the model says `camo` OR the hardware class.**
  - Group A: `labels_detailed` class = camo (523). The hardware class comes from the fine label: ICV/Tank → tank, SPATracked → arty,
    TransportVehicle/LTV → vehicle, HQ9/MissileSystem/HQ22 → missile, Type780/RadarSystem → radar. The fine label `Camo` (196) stays `camo`.
  - Group B: P_Camo = Yes with the real class kept (227). Accepted: the real class or `camo`.
- **Recall row = the hardware class.** A camouflaged HQ9 counts in the missile row. Only fine label `Camo` stays in the camo row.
- **Matching:** boxes in score order. Each takes the best-IoU unmatched object whose accepted classes include the box class.
  One box ↔ one object. Precision is per predicted class.

## Result — detailed + camo credit

| class | deploy thr | GT | TP | FP | precision % | recall % | F1 % |
|---|--:|--:|--:|--:|--:|--:|--:|
| radar | 0.39 | 208 | 118 | 112 | 50.0 | 56.7 | 53.2 |
| missile | 0.85 | 319 | 172 | 62 | 72.1 | 53.9 | 61.7 |
| arty | 0.86 | 245 | 100 | 93 | 40.8 | 40.8 | 40.8 |
| logistic | 0.77 | 241 | 60 | 138 | 30.3 | 24.9 | 27.3 |
| camo | 0.42 | 196 | 119 | 330 | 44.4 | 60.7 | 51.3 |
| smallvehicle | 0.24 | 822 | 502 | 714 | 41.3 | 61.1 | 49.3 |
| tank | 0.54 | 966 | 656 | 129 | 81.7 | 67.9 | 74.2 |
| vehicle | 0.84 | 2103 | 1560 | 840 | 64.9 | 74.2 | 69.2 |
| **MICRO** | | **5100** | **3287** | **2418** | **57.6** | **64.5** | **60.8** |
| **MACRO** | | | | | **53.2** | **55.0** | |

Credit used: 145 objects were matched by a `camo` box sitting on hardware (counted in the hardware row).
By group (objects / matched): plain 2795/4350 · A_camo_hw 236/327 · B_pcamo 137/227 · A_camo_class 119/196.

## For comparison — strict labels_detailed (no camo credit)

| class | deploy thr | GT | TP | FP | precision % | recall % | F1 % |
|---|--:|--:|--:|--:|--:|--:|--:|
| radar | 0.39 | 206 | 113 | 111 | 50.4 | 54.9 | 52.6 |
| missile | 0.85 | 307 | 160 | 62 | 72.1 | 52.1 | 60.5 |
| arty | 0.86 | 211 | 64 | 93 | 40.8 | 30.3 | 34.8 |
| logistic | 0.77 | 241 | 60 | 138 | 30.3 | 24.9 | 27.3 |
| camo | 0.42 | 523 | 249 | 345 | 41.9 | 47.6 | 44.6 |
| smallvehicle | 0.24 | 822 | 502 | 714 | 41.3 | 61.1 | 49.3 |
| tank | 0.54 | 737 | 483 | 221 | 68.6 | 65.5 | 67.0 |
| vehicle | 0.84 | 2053 | 1536 | 854 | 64.3 | 74.8 | 69.1 |
| **MICRO** | | **5100** | **3167** | **2538** | **55.5** | **62.1** | **58.6** |
| **MACRO** | | | | | **51.2** | **51.4** | |

## For comparison — simple mapping (previous report, `oct_w2/n0001`)

| class | GT | precision % | recall % |
|---|--:|--:|--:|
| radar | 176 | 41.1 | 52.3 |
| missile | 162 | 30.2 | 41.4 |
| arty | 179 | 40.8 | 35.8 |
| logistic | 241 | 30.3 | 24.9 |
| camo | 750 | 44.4 | 35.2 |
| smallvehicle | 822 | 41.3 | 61.1 |
| tank | 729 | 67.5 | 65.2 |
| vehicle | 2041 | 64.2 | 75.2 |
| **MICRO** | **5100** | **53.6** | **60.0** |

Note: n0001 used the reference per-class first-match rule, and this report uses a global best-IoU match. On the same GT the two
differ by a few objects. The large differences come from the GT definition.
