# EXP-2026-10-26-02-best-pred-files-guide — AI-authored

Author: AI agent (Claude). Generated: 2026-10-26T19:26:54+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-26-02-best-pred-files-guide`

Status: done (analysis only, no GPU).

## Request summary and authorization

2026-10-26 (IST). The researcher asked for a document to compare the best prediction files for all test images. It should list every required file, where to find what, the formats, and our findings. Analysis only.

## What was done

- **Guide:** [knowledge/test_predictions_guide_ai.md](../../../../../knowledge/test_predictions_guide_ai.md).
- **Built** with `code/build_best.py` (env `/sfs/envs/yolo_models`):
  - `preds/best_per_class/labels/<scene>.txt`: for each class, the boxes of the best model for that class (P04 v2, IoU 0.3, best F1) at score ≥ its confidence;
  - `gt_p04v2/labels/<scene>.txt`: GT on the 0.30 m grid with the P04 v2 class, instance ID and official class;
  - `reports/per_image.tsv`: per image × class GT / TP / FP / FN for best_per_class and for m0001 deploy.
- All of these are under `/sfs/stage2/artifacts/EXP-2026-10-26-02-best-pred-files-guide/`.

## Results

- **best_per_class re-scored:** MICRO P 63.3 / R 63.9 / F1 63.6, MACRO 55.0 (TP 3,257, FP 1,892, GT 5,100). This equals the EXP-2026-10-26-01 best-per-class result.
- **m0001 deploy under the same GT:** P 56.4 / R 63.1 / F1 59.6, MACRO 51.5.
- **Best single prediction set under P04 v2:** EXP-2026-10-14-14 at 63.4 / 54.3 (m0001: 63.0 / 54.0).

Evidence: [per_image_ai.tsv](per_image_ai.tsv), [best_per_class_check_ai.json](best_per_class_check_ai.json).

## Conclusions and next steps

- **Caveat:** best_per_class is an upper bound that mixes 4 models. The model and confidence for each class were picked on test_80.
- **Next:** compare images visually using best_per_class vs m0001 deploy against gt_p04v2 (same coordinates).
