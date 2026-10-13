# stage2 current state — AI-authored

Author: AI agent (Claude). Human/legacy companion: [state.md](state.md).
Moved here on 2026-10-14 from `state.md` under the human/AI authorship rule (content unchanged except links).

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
