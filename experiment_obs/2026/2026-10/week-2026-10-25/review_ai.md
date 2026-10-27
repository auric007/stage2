# Week 2026-10-25 — review (AI-authored)

Author: AI agent (Claude). Human companion: [review.md](review.md).

## Best metrics so far — per-class combination (saved 2026-10-27)

- **Scope:** test_80 (Test1 43 + Test2 37), IoU 0.3.
- **Class mapping:** P04 v2 (one class per object, 15 attribute rules; `eval/protocols/P04_single_class_remap/rules_v2.json`).
- **Source:** [EXP-2026-10-27-01](experiments/EXP-2026-10-27-01-best-combination-v1/record_ai.md).
- **How it's built and re-run:** [knowledge/best_combination_ai.md](../../../knowledge/best_combination_ai.md).

| class | GT | Precision | Recall | F1 | TP / FP / FN | model used | conf ≥ |
|---|--:|--:|--:|--:|---|---|--:|
| arty | 162 | 42.1 | 37.7 | 39.7 | 61 / 84 / 101 | y8m D-frozen + y11l ensemble (EXP-2026-10-14-14) | 0.90 |
| camo | 548 | 50.8 | 49.3 | 50.0 | 270 / 262 / 278 | y8m D-frozen + y11l ensemble (EXP-2026-10-14-14) | 0.80 |
| logistic | 241 | 26.7 | 30.7 | 28.6 | 74 / 203 / 167 | YOLO11l fine-tuned on D (EXP-2026-10-14-13) | 0.60 |
| missile | 197 | 66.7 | 61.9 | 64.2 | 122 / 61 / 75 | y8m D-frozen + y11l ensemble (EXP-2026-10-14-14) | 0.90 |
| radar | 190 | 71.4 | 50.0 | 58.8 | 95 / 38 / 95 | original YOLO11l alone (EXP-2026-10-14-17) | 0.60 |
| smallvehicle | 822 | 62.5 | 49.4 | 55.2 | 406 / 244 / 416 | YOLO11l fine-tuned on D (EXP-2026-10-14-13) | 0.50 |
| tank | 744 | 70.2 | 70.4 | 70.3 | 524 / 222 / 220 | m0001 ensemble | 0.90 |
| vehicle | 2196 | 68.7 | 77.6 | 72.9 | 1705 / 778 / 491 | m0001 ensemble | 0.90 |
| **MICRO** | 5100 | 63.3 | 63.9 | **63.6** | 3257 / 1892 / 1843 | | |
| **MACRO** | | 57.4 | 53.4 | **55.0** | | | |

Notes:
- **With cross-class dedup** (one box per place, the product output): MICRO 62.7 / MACRO 54.6.
- **For comparison:**
  - best single prediction set EXP-2026-10-14-14: 63.4 / 54.3;
  - m0001 alone: 63.0 / 54.0;
  - official P01 scoring, best per class: 60.2 / 49.6.
- **Upper bound:** model and confidence per class were chosen on test_80 itself.
