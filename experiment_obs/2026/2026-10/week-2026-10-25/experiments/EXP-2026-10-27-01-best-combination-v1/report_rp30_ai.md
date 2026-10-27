# Best-per-class combination — test_80, IoU 0.3, objective rp30

Mapping: `/sfs/stage2/eval/protocols/P04_single_class_remap/rules_v2.json` · runs scored: 21 · built by `code/engine_v1/combine_best.py`

| class | GT | run picked | conf ≥ | P | R | F1 | TP / FP / FN | after dedup P / R / F1 |
|---|--:|---|--:|--:|--:|--:|---|---|
| arty | 162 | m0001 ensemble (y8m+y11l) | 0.80 | 33.7 | 40.7 | 36.9 | 66 / 130 / 96 | 34.6 / 40.1 / 37.1 |
| camo | 548 | EXP-2026-10-14-14-ens-dfrozen-y11l | 0.30 | 32.0 | 63.1 | 42.5 | 346 / 736 / 202 | 39.5 / 44.5 / 41.9 |
| logistic | 241 | EXP-2026-10-14-16-ens-d-full-y8md-y11ld | 0.80 | 30.5 | 25.3 | 27.7 | 61 / 139 / 180 | 30.6 / 24.9 / 27.5 |
| missile | 197 | EXP-2026-10-14-16-ens-d-full-y8md-y11ld | 0.40 | 34.0 | 72.6 | 46.3 | 143 / 278 / 54 | 38.2 / 70.1 / 49.5 |
| radar | 190 | EXP-2026-10-14-17-y11l-single-baseline | 0.05 | 30.4 | 68.4 | 42.1 | 130 / 298 / 60 | 30.4 / 68.4 / 42.1 |
| smallvehicle | 822 | EXP-2026-10-14-15-ens-r-y8mft-y11l | 0.10 | 31.1 | 68.2 | 42.7 | 561 / 1244 / 261 | 33.0 / 61.1 / 42.9 |
| tank | 744 | m0001 ensemble (y8m+y11l) | 0.05 | 37.7 | 91.9 | 53.5 | 684 / 1131 / 60 | 47.8 / 64.7 / 54.9 |
| vehicle | 2196 | EXP-2026-10-14-16-ens-d-full-y8md-y11ld | 0.10 | 31.2 | 93.5 | 46.8 | 2053 / 4520 / 143 | 34.4 / 80.3 / 48.2 |
| **combined MICRO** | 5100 | | | 32.3 | 79.3 | **45.9** | | |
| **combined MACRO** | | | | 32.6 | 65.5 | **42.3** | | |
| **combined + dedup MICRO** | 5100 | | | 35.8 | 66.3 | **46.5** | | |
| **combined + dedup MACRO** | | | | 36.1 | 56.8 | **43.0** | | |

Top single runs (MACRO F1 at each class's best conf):

- EXP-2026-10-14-14-ens-dfrozen-y11l: 54.3
- m0001 ensemble (y8m+y11l): 54.0
- EXP-2026-10-14-08-ens-d-y8mft-y11l: 53.7
- EXP-2026-10-14-16-ens-d-full-y8md-y11ld: 53.6
- EXP-2026-10-14-06-ens-b-y8mft-y11l: 53.3
