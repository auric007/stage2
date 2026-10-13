# stage2 current state

As of 2026-10-13, Asia/Kolkata.

- Notebook hierarchy created in the existing stage2 folder.
- Local research context is documented in `/sfs/stage2/AGENTS.md` and `STATE.md`.
  This transport setup did not validate model claims or inspect active research runs.
- No research experiments were run or results produced by notebook setup.
- Local repository configured for https://github.com/auric007/stage2, branch main,
  on 2026-10-14. Server import and a guarded notebook push/pull were verified.
- Server notebook imported at `/sfs/stage2/experiment_obs` on 2026-10-14; separate
  notebook Git metadata protects the research tree. A 30-second sync cycle is
  configured and running; a fresh live cycle succeeded at about 01:20 IST.
- Research source directory is `/sfs/stage2`; its root AGENTS.md and STATE.md
  describe the local research. Existing model claims were not re-evaluated here.
- Next proposed action: define the stage2 research objective and first weekly task.

Evidence: [setup daily record](2026/2026-10/week-2026-10-11/daily/2026-10-13.md).
See [weekly state](2026/2026-10/week-2026-10-11/state.md) and [connection handoff](mac-handoff.md).

## Research position — 2026-10-14 01:45 IST (Claude; supersedes "no research results recorded" above)

- **Best model:** m0001 = sep_w1 ensemble (YOLOv8m-OBB + YOLO11l-OBB, TTA), weights `/sfs/stage2/models/BEST`
  ([EXP-2026-09-01-01](2026/2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record.md)).
- **Metrics on test_80 (IoU 0.3):** P01 deploy P 53.6 / R 60.0 · P01 recall-first MICRO 74.4 / MACRO 58.0 · P02 P 57.6 / R 64.4 ·
  P03 missile row R 67.1 / P 72.1 (radar row pending).
- **Active:** [EXP-2026-10-12-01](2026/2026-10/week-2026-10-11/experiments/EXP-2026-10-12-01-attr-pattern-merge/record.md) — user rule set;
  next decision: radar (EWS → vehicle? stowed Closed/Nil → vehicle? helicopter boxes neutral?).
- **Where things are:** this notebook = all records/analysis; `/sfs/stage2/artifacts/<EXP-ID>/` = code and large outputs;
  `/sfs/stage2/models`, `/sfs/stage2/data/derived`, `/sfs/stage2/eval` (test_80 + protocol definitions).
- Decisions: [knowledge/decisions.md](knowledge/decisions.md); findings: [knowledge/findings.md](knowledge/findings.md); open ideas/checks: [knowledge/ideas.md](knowledge/ideas.md).
