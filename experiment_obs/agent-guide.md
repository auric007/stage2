# stage2 research agent guide

## Human and AI authorship — effective 2026-10-14

This rule applies recursively to every directory under each registered `experiment_obs/`,
including root/knowledge notes, month/week notes, daily entries, experiment records,
reports, attachments, templates and future subdirectories. It supersedes earlier
unsuffixed AI write destinations in this guide and linked instructions.

- **Human lane:** unsuffixed files such as `plan.md`, `state.md`, `review.md`,
  `YYYY-MM-DD.md`, and `record.md` are researcher/Obsidian-editable source notes.
  Preserve their contents; never append AI commentary or results to them.
- **AI lane:** write AI-authored text, results, analysis, summaries, plans, state,
  decisions, reviews and new generated notebook evidence to a same-directory,
  same-base-name counterpart with lowercase **`_ai` immediately before the extension**:
  `plan_ai.md`, `state_ai.md`, `review_ai.md`, `YYYY-MM-DD_ai.md`, `record_ai.md`,
  `metrics_ai.json`, `plot_ai.png`. With no extension, end the name in `_ai`.
  Use exactly one lowercase suffix, not `_AI` or `_ai_ai`.
- Create the pair when actually writing a note: keep an existing human file intact;
  if absent, create only an **empty human slot**, not fabricated researcher text.
  Start the AI note with an authorship marker and a link to its human counterpart.
  Empty slots are not evidence of activity or experiment completion.
- Read the human note, relevant legacy evidence and the `_ai` counterpart before work.
  Reference the source note/previous experiment, record timestamps and stable task IDs,
  append observed outcomes and uncertainty, and distinguish proposals from authorization.
  AI notes summarize or respond to the human plan; they do not replace it.
- Keep daily AI results under the actual Asia/Kolkata date in the Sunday-based week:
  `YYYY/YYYY-MM/week-YYYY-MM-DD/daily/YYYY-MM-DD_ai.md`.
  Keep actual experiment setup/results/analysis in that experiment's `record_ai.md`
  and AI-suffixed supporting files; link them from the daily AI note.
  Planning alone still does not create an experiment or authorize running it.
- Obsidian/server synchronization retains these filenames and roles. A human may
  review/edit an `_ai` file, but that does not silently turn its content into a
  researcher-authored unsuffixed file. Copy/adopt material only on explicit request.
- Existing unsuffixed and mixed-author history stays at its current path and remains
  readable evidence. Do not bulk rename it, claim it was wholly written by the human,
  or break historical links. Apply the new split to every future substantive AI write.
- **Fixed-name control exceptions:** `AGENTS.md`, `CLAUDE.md`, `AGENTS.override.md`,
  and `agent-guide.md` retain loader-required names. Their policy edits are allowed
  when authorized; they are instructions, not experiment results or human notes.
  Existing machine-consumed input/protocol/config files keep their required names;
  this rule alone does not authorize editing them. New AI-generated notebook copies
  and reports still use `_ai`. Code, weights and large outputs outside the notebook
  keep their project's normal naming and publication boundaries.

Use `templates/*_ai.md` for AI records. Registries and indexes created/updated by AI
inside the notebook use corresponding `_ai` files; read both lanes when allocating
IDs or collecting evidence so historical records are not missed or double-counted.
No sync watcher or autonomous research execution is added by this rule.

## Read before work

1. Read [state](state.md), [long-term plan](plan.md), and the current month/week plans.
2. Read relevant [findings](knowledge/findings.md), [decisions](knowledge/decisions.md),
   and [ideas](knowledge/ideas.md), including superseded entries.
3. Read relevant recent daily records and previous weekly/monthly reviews if present.
   Follow links to the original experiment evidence; missing history is unknown.
4. Verify the actual working directory, project/code repository and applicable
   agent instructions. This is stage2; do not import other projects' results as its evidence.

## Record substantive requests automatically when this guide is loaded

At task start, append a brief intent summary, stable task ID, date/time in
Asia/Kolkata, working directory, and authorized scope to today's daily record.
Append meaningful follow-ups and observed outcomes under the same task ID.
Record actual changed paths, relevant executed commands, what worked, what failed,
known or suspected causes, unknowns, evidence links and the next proposed action.
Use concise summaries; retain exact commands or parameters when they are evidence.
Do not record credentials, hidden/private reasoning, or conversations you cannot access.
Honor explicit read-only/no-file-edit instructions.

Before finishing, update the applicable experiment record and weekly state.
Refresh month/root state only when their summary changes. Report which notes were
saved and any blocker. Saved, committed, pushed and pulled are separate states;
claim each only when verified. Preserve prior entries and concurrent local edits.

## Experiments and accumulated knowledge

Create an EXP-YYYY-MM-DD-NN-description directory only for an actual tracked
experiment. Allocate a unique ID and use [the experiment template](templates/experiment_ai.md).
Record the actual code repository/SHA and dirty state separately from notebook
revision, configuration, environment, data/model versions, seeds, commands, metrics,
failures, artifact paths and conclusions. Mark unobserved results unknown.
Keep large artifacts external with paths/hashes; attach only relevant reviewed evidence.
Planning-only conversations need no fabricated experiment.

Promote supported observations into findings; keep untested possibilities in ideas.
Record decisions with evidence, tradeoffs and revisit criteria. Preserve superseded
claims and decisions with links to their replacements. Repeated summaries do not
constitute independent evidence.

## Dates and reviews

Use Sunday-start weeks in Asia/Kolkata, filed under the starting Sunday's month.
Keep ongoing experiments in their original folders. Reviews link original evidence,
compare like-for-like runs, include failures and missing coverage, and propose next
steps. Month reviews include actual dates from all overlapping weeks, even those
stored under a neighboring month/year. No scheduled review generator is installed.

## Activation and execution

An agent must explicitly load this guide through its supported instruction entry
point or a startup request. This file alone does not activate logging or a watcher.
Do not launch experiments merely because plans or notes exist. Execute only work
already authorized by the user. Never invent results or silently overwrite conflicts.
As of 2026-10-14, this server project is registered at `/sfs/stage2`, notebook
`/sfs/stage2/experiment_obs`, remote `auric007/stage2`, branch main. Read the
shared `/sfs/research-notebook/logging-policy.md` and root research instructions.
Notebook Git metadata is `/sfs/markdown-sync/git/stage2`, separate from code Git.
Since 2026-10-14 (user decision D-20261014-01, `knowledge/decisions_ai.md`) this notebook is the ONLY stage2
tracking location: Sunday weeks and EXP-YYYY-MM-DD-NN IDs apply to all stage2 work; the old `/sfs/stage2/journal`,
`registry` and `experiments` were migrated here and removed (pre-migration IDs e0001-e0007 survive as cross-references).
Code, weights and large outputs stay outside in `/sfs/stage2/artifacts/<EXP-ID>/`, `models/`, `data/`.
Record actual code revision separately from notebook revision.
Only experiment_obs is synchronized, with a 30-second wait after each cycle.
Check controller status for observed success; never infer execution from a note.
