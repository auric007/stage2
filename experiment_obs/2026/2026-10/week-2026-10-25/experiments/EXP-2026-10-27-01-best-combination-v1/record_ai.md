# EXP-2026-10-27-01-best-combination-v1 — AI-authored

Author: AI agent (Claude). Generated: 2026-10-27T15:28:59+05:30.
Human companion: [record.md](record.md) (researcher's notes; empty until they write).

Artifacts (outside the notebook, not synced): `/sfs/stage2/artifacts/EXP-2026-10-27-01-best-combination-v1`

Status: done (analysis only, no GPU).

## Request summary and authorization

2026-10-27 (IST). Researcher: the product goal is the best metrics; any mix of models or experiments per class is fine. They asked for a doc explaining how the best number is obtained and combined, repeatable for future experiments. Analysis only.

## What was done

- **New reusable tool** `/sfs/stage2/code/engine_v1/combine_best.py`:
  - discovers all prediction sets;
  - scores them under a mapping (default P04 v2);
  - picks the best set + confidence per class;
  - writes the combined and the combined + dedup prediction folders, a report, a leaderboard and a per-image table.
- **P04 v2 rules frozen** as `/sfs/stage2/eval/protocols/P04_single_class_remap/rules_v2.json` (+ README, protocols index).
- **Doc:** [knowledge/best_combination_ai.md](../../../../../knowledge/best_combination_ai.md).
- **Run:** `combine_best.py --out artifacts/EXP-2026-10-27-01-best-combination-v1/{f1,rp30}`, 21 prediction sets, about 23 s each.

## Results (test_80, P04 v2, IoU 0.3)

- **Best F1 per class:** combined MICRO F1 63.6 / MACRO 55.0. This equals EXP-2026-10-26-01/-02, so the tool reproduces the earlier result.
- **With cross-class dedup:** 62.7 / 54.6.
- **Recall-first:** combined MICRO R 79.3 (P 32.3), MACRO R 65.5.
- **Best single set:** EXP-2026-10-14-14 at 54.3 MACRO; m0001 at 54.0.

Evidence:
- [report_f1_ai.md](report_f1_ai.md), [report_rp30_ai.md](report_rp30_ai.md);
- [picks_f1_ai.json](picks_f1_ai.json), [leaderboard_ai.tsv](leaderboard_ai.tsv), [per_image_ai.tsv](per_image_ai.tsv).

## Conclusions and next steps

- **Product recipe:** 4 weight files and 6 model passes per image (doc section 4). A single inference wrapper for new images is not built yet.
- **Caveat:** the picks are made on test_80, so this is an upper bound until a validation split exists.
- **Going forward:** re-run `combine_best.py` after every new experiment and update the doc's section 2.
