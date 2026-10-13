# stage2 research notebook — agent navigation (AI-authored)

Author: AI agent (Claude). Generated: 2026-10-14. Human companion: [README.md](README.md).

The researcher's [README](README.md) is the human index. Since 2026-10-14 agents follow the authorship rule in
[agent-guide.md](agent-guide.md): unsuffixed files are the researcher's; agents write the same-name `_ai` files.

| Information | Researcher (human lane) | Agents (`_ai` lane) |
|---|---|---|
| Project position | [state.md](state.md) | [state_ai.md](state_ai.md) |
| Long-term plan | [plan.md](plan.md) | `plan_ai.md` (proposals only) |
| Daily work | `YYYY/YYYY-MM/week-<Sunday>/daily/YYYY-MM-DD.md` | `…/daily/YYYY-MM-DD_ai.md` |
| Experiment | `…/experiments/EXP-…/record.md` | `…/experiments/EXP-…/record_ai.md` + `*_ai.*` evidence |
| Week / month | `plan.md`, `state.md`, `review.md` | `plan_ai.md`, `state_ai.md`, `review_ai.md` |
| Knowledge | `knowledge/{findings,decisions,ideas}.md` | `knowledge/{findings,decisions,ideas}_ai.md` |

Start an experiment with `/sfs/stage2/code/new_experiment.sh <slug>` (creates the empty `record.md`, the `record_ai.md`,
the daily `_ai` line and the non-synced `/sfs/stage2/artifacts/<EXP-ID>/`).

## Experiments so far
- [EXP-2026-09-01-01 final ensemble (m0001)](2026/2026-08/week-2026-08-30/experiments/EXP-2026-09-01-01-final-ensemble-train/record_ai.md)
- [EXP-2026-09-24-01 sensor selection](2026/2026-09/week-2026-09-20/experiments/EXP-2026-09-24-01-sensor-exp-selection/record_ai.md) ·
  [EXP-2026-09-24-02 top-k disagreement](2026/2026-09/week-2026-09-20/experiments/EXP-2026-09-24-02-topk-disagree-export/record_ai.md)
- [EXP-2026-10-05-01 JSON export](2026/2026-10/week-2026-10-04/experiments/EXP-2026-10-05-01-test-pred-05-export/record_ai.md) ·
  [EXP-2026-10-08-01 per-image matrices](2026/2026-10/week-2026-10-04/experiments/EXP-2026-10-08-01-imagewise-deploy/record_ai.md) ·
  [EXP-2026-10-09-01 camo credit](2026/2026-10/week-2026-10-04/experiments/EXP-2026-10-09-01-detailed-camo-credit/record_ai.md)
- [EXP-2026-10-12-01 attribute patterns → user rules (running)](2026/2026-10/week-2026-10-11/experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md)
