# Weekly state (week of 2026-11-22) — AI-authored

Author: AI agent (Claude). Human companion: [state.md](state.md).

Updated: 2026-11-23 19:25 IST. Task: `TASK-20261123-lowres-poc-split`.

- **New train/test split adopted for the next training:** `/sfs/data/lowres_poc` (train 836 = 420 LR + 416 new HR; test 136 LR at GSD 0.58–0.80).
- Labels cleaned per the researcher's rules (D-20261123-01): 9 trained classes incl. spa; 70,019 objects. Details: [2026-11-23_ai.md](daily/2026-11-23_ai.md).
- No training, model, protocol or deployment change. The stage-2 tool stays on HYBRID v2.
- Open (researcher): validation split, > 0.8 m scenes, training GSD; then tiling.
