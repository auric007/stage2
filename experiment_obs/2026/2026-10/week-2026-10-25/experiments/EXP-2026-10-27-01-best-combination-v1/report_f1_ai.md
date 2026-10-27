# Best-per-class combination — test_80, IoU 0.3, objective f1

Mapping: `/sfs/stage2/eval/protocols/P04_single_class_remap/rules_v2.json` · runs scored: 21 · built by `code/engine_v1/combine_best.py`

| class | GT | run picked | conf ≥ | P | R | F1 | TP / FP / FN | after dedup P / R / F1 |
|---|--:|---|--:|--:|--:|--:|---|---|
| arty | 162 | EXP-2026-10-14-14-ens-dfrozen-y11l | 0.90 | 42.1 | 37.7 | 39.7 | 61 / 84 / 101 | 43.9 / 37.7 / 40.5 |
| camo | 548 | EXP-2026-10-14-14-ens-dfrozen-y11l | 0.80 | 50.8 | 49.3 | 50.0 | 270 / 262 / 278 | 57.6 / 42.3 / 48.8 |
| logistic | 241 | EXP-2026-10-14-13-y11l-finetune-d | 0.60 | 26.7 | 30.7 | 28.6 | 74 / 203 / 167 | 26.8 / 30.7 / 28.6 |
| missile | 197 | EXP-2026-10-14-14-ens-dfrozen-y11l | 0.90 | 66.7 | 61.9 | 64.2 | 122 / 61 / 75 | 67.8 / 61.9 / 64.7 |
| radar | 190 | EXP-2026-10-14-17-y11l-single-baseline | 0.60 | 71.4 | 50.0 | 58.8 | 95 / 38 / 95 | 71.4 / 50.0 / 58.8 |
| smallvehicle | 822 | EXP-2026-10-14-13-y11l-finetune-d | 0.50 | 62.5 | 49.4 | 55.2 | 406 / 244 / 416 | 63.4 / 47.9 / 54.6 |
| tank | 744 | m0001 ensemble (y8m+y11l) | 0.90 | 70.2 | 70.4 | 70.3 | 524 / 222 / 220 | 77.1 / 61.6 / 68.5 |
| vehicle | 2196 | m0001 ensemble (y8m+y11l) | 0.90 | 68.7 | 77.6 | 72.9 | 1705 / 778 / 491 | 71.3 / 73.1 / 72.2 |
| **combined MICRO** | 5100 | | | 63.3 | 63.9 | **63.6** | | |
| **combined MACRO** | | | | 57.4 | 53.4 | **55.0** | | |
| **combined + dedup MICRO** | 5100 | | | 66.1 | 59.6 | **62.7** | | |
| **combined + dedup MACRO** | | | | 59.9 | 50.7 | **54.6** | | |

Top single runs (MACRO F1 at each class's best conf):

- EXP-2026-10-14-14-ens-dfrozen-y11l: 54.3
- m0001 ensemble (y8m+y11l): 54.0
- EXP-2026-10-14-08-ens-d-y8mft-y11l: 53.7
- EXP-2026-10-14-16-ens-d-full-y8md-y11ld: 53.6
- EXP-2026-10-14-06-ens-b-y8mft-y11l: 53.3
