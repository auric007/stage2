# stage2 current state — AI-authored

Author: AI agent (Claude). Human/legacy companion: [state.md](state.md).
Moved here on 2026-10-14 from `state.md` under the human/AI authorship rule (content unchanged except links).

## Update — 2026-10-26 16:36 IST (Claude)

- **P04 v2 (draft):** P04 plus camouflaged radar → radar (D-20261026-01). Best per class MICRO 63.6 / MACRO 55.0; m0001 alone 63.0 / 54.0.
- **Camouflaged missiles:** better under missile than under camo, by a wide margin.
- Source: [EXP-2026-10-26-01](2026/2026-10/week-2026-10-25/experiments/EXP-2026-10-26-01-pcamo-missile-radar-mapping/record_ai.md).

## Research position — 2026-10-23 19:25 IST (Claude; supersedes the 2026-10-14 block below)

- **Best single model:** still m0001. Of the week's fine-tunes, the original YOLO11l alone is the best all-round ([week review](2026/2026-10/week-2026-10-11/review_ai.md)).
- **Reporting protocol:** P03 is retired for reporting (D-20261023-01). The draft **P04** gives each object one class via 14 attribute rules.
- **Metrics on test_80 (IoU 0.3, under P04):**
  - best per class: MICRO F1 63.5, MACRO 54.4, R@P≥0.30 79.2 / 64.9;
  - m0001 alone: 63.0 / 53.8;
  - official P01, best per class, for comparison: 60.2 / 49.6.
  - Source: [EXP-2026-10-23-02](2026/2026-10/week-2026-10-18/experiments/EXP-2026-10-23-02-single-class-attr-remap/record_ai.md).
  - These are upper bounds: the rules and picks were made on test_80.
- **Open:**
  - the researcher's review of the 14 rules;
  - a held-out split to confirm them;
  - assigning the 15_10 data.

## Research position — 2026-10-14 ~01:34 IST (Claude; supersedes "no research results recorded" above)

- **Best model:** m0001 = sep_w1 ensemble (YOLOv8m-OBB + YOLO11l-OBB, TTA), weights `/sfs/stage2/models/BEST`
  ([EXP-2026-09-01-01](2026/2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record_ai.md)).
- **Metrics on test_80 (IoU 0.3):** P01 deploy P 53.6 / R 60.0 · P01 recall-first MICRO 74.4 / MACRO 58.0 · P02 P 57.6 / R 64.4 ·
  P03 missile row R 67.1 / P 72.1 (radar row pending).
- **Active:** [EXP-2026-10-12-01](2026/2026-10/week-2026-10-11/experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md) — user rule set;
  next decision: radar (EWS → vehicle? stowed Closed/Nil → vehicle? helicopter boxes neutral?).
- **Where things are:** this notebook = all records/analysis; `/sfs/stage2/artifacts/<EXP-ID>/` = code and large outputs;
  `/sfs/stage2/models`, `/sfs/stage2/data/derived`, `/sfs/stage2/eval` (test_80 + protocol definitions).
- Decisions: [knowledge/decisions.md](knowledge/decisions_ai.md); findings: [knowledge/findings.md](knowledge/findings_ai.md); open ideas/checks: [knowledge/ideas.md](knowledge/ideas_ai.md).
