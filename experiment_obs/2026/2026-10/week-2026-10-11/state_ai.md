# Weekly state — AI documentation update

Author: AI agent (Codex). Generated: 2026-10-14T01:48:11+05:30.
Human/legacy companion: [state.md](state.md).

As of 2026-10-14T01:48:11+05:30. Task RN-20261014-ai-authorship.

This update concerns notebook authorship only. The shared naming policy and
local guides/templates now require lowercase `_ai` for AI-generated notes.
Historical research evidence stays in its original files; no model or experiment status
has been re-evaluated in this task.

Evidence: [today's AI daily note](daily/2026-10-14_ai.md) and [notebook instructions](../../../AGENTS.md).

## Verification complete

As of 2026-10-14T01:50:54+05:30: 30 helper tests passed; recursive rules/template pairs verified; existing human/history notes preserved. This is a documentation/infrastructure outcome, not a research experiment result. See [daily evidence](daily/2026-10-14_ai.md).

<!-- moved from state.md on 2026-10-14 (authorship split) -->
## Update — 2026-10-14 01:37 IST (Claude)

- Stage2 tracking moved into this notebook (decision D-20261014-01): all past work is now 7 EXP records (Sept 1 – Oct 12);
  code and large outputs in `/sfs/stage2/artifacts/<EXP-ID>/`.
- Active: [EXP-2026-10-12-01](experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md) — P03 missile row locked; radar decisions pending.

## Update — 2026-10-14 02:13 IST (Claude)

- Overnight GPU queue running (V100): baseline → missile relabel B, C, D (10-epoch fine-tunes of m0001 y8m) → control. Records:
  EXP-2026-10-14-01 … -05 in [experiments/](experiments/README_ai.md); progress in [daily AI log](daily/2026-10-14_ai.md).
- Pending researcher decisions: radar rule row (EWS / stowed / helicopters).

## Update — 2026-10-16 15:48 IST (Claude)

- Week closed for research by the researcher (no further analysis this week). Final summary: [review_ai.md](review_ai.md) "Week summary — final".
- Best all-round: original YOLO11l alone (deploy F1 57.8); best missiles: YOLO11l on D (missile F1 71.5). GPU queue empty since 2026-10-15 03:12.
