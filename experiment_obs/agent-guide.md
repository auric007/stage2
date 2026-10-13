# stage2 research agent guide

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
experiment. Allocate a unique ID and use [the experiment template](templates/experiment.md).
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
Stage2 synchronization and server routing remain unconfigured; do not reuse another
project's credentials, remote or server path without confirmation of the mapping.
