# stage2 research notebook

Start with [current state](state.md), [long-term plan](plan.md), and the
[agent guide](agent-guide.md). This notebook follows the same structure as the
nanoQwenVL and orchestration notebooks, with independent project evidence.

## Current work

- [October plan](2026/2026-10/plan.md), [state](2026/2026-10/state.md), [review](2026/2026-10/review.md).
- [Week of October 11 plan](2026/2026-10/week-2026-10-11/plan.md), [state](2026/2026-10/week-2026-10-11/state.md), [review](2026/2026-10/week-2026-10-11/review.md).
- [October 13 daily record](2026/2026-10/week-2026-10-11/daily/2026-10-13.md).
- [Ideas](knowledge/ideas.md), [findings](knowledge/findings.md), [decisions](knowledge/decisions.md).

## Where to write

| Information | Location |
| --- | --- |
| Long-term objectives | Root plan.md |
| Latest verified project position | Root state.md |
| Month/week priorities and progress | That period's plan.md and state.md |
| Brief requests, actual work and outcomes | That week's daily/YYYY-MM-DD.md |
| Experiment setup, commands, results and evidence | That week's experiments/EXP-YYYY-MM-DD-NN-description/record.md |
| Evidence-backed lessons and choices | knowledge/findings.md and decisions.md |
| Weekly/monthly synthesis | That period's review.md |
| Reusable record layouts | templates/ |
| Preserved old material | legacy/ |
| GitHub/server connection details | mac-handoff.md |

## Calendar and records

Use Asia/Kolkata. Weeks start Sunday at 00:00 and end before the next Sunday.
Keep a whole week under its starting Sunday's month/year. Monthly reviews count
actual work dates, including relevant days in neighboring months' week folders.
Create only needed periods. Keep an experiment in its original folder across weeks.
Use relative links so the notebook remains portable between Obsidian and GitHub.

Summarize substantive requests rather than copying full prompts by default.
Record what worked, what failed, evidence, uncertainty and the next proposed step.
Saving a plan does not authorize experiment execution.

## Connection status

- GitHub repository: https://github.com/auric007/stage2
- Branch: main. Company identity: Ashutosh Bharti <ashutosh.bharti@auricai.in>.
- Local Git repository configured on 2026-10-14 for this notebook only.
- Initial notebook publication was fetched successfully to the server on 2026-10-14.
- Server notebook: `/sfs/stage2/experiment_obs`. Pinned company sync is configured
  for a 30-second wait after each cycle; inspect status before claiming it is running.
- Notebook Git metadata: `/sfs/markdown-sync/git/stage2`, separate from research
  code. Root instructions point to this guide; original research conventions remain.

The notebook boundary is experiment_obs/ recursively, including ordinary nested
files and dotfiles. Research code stays outside the Obsidian checkout. See the
[connection handoff](mac-handoff.md). Authentication is repository-scoped in macOS
Keychain; no token belongs in this notebook.
