# Weekly state (week of 2026-11-22) — AI-authored

Author: AI agent (Claude). Human companion: [state.md](state.md).

Updated: 2026-11-23 19:25 IST. Task: `TASK-20261123-lowres-poc-split`.

- **New train/test split adopted for the next training:** `/sfs/data/lowres_poc` (train 836 = 420 LR + 416 new HR; test 136 LR at GSD 0.58–0.80).
- Labels cleaned per the researcher's rules (D-20261123-01): 9 trained classes incl. spa; 70,019 objects. Details: [2026-11-23_ai.md](daily/2026-11-23_ai.md).
- No training, model, protocol or deployment change. The stage-2 tool stays on HYBRID v2.
- Open (researcher): validation split, > 0.8 m scenes, training GSD; then tiling.
- 2026-11-23 20:15 IST: **training running** on lowres_poc — [EXP-2026-11-23-01](experiments/EXP-2026-11-23-01-lowres-poc-train/record_ai.md) (y8m → y11l → TTA test scoring; ETA ~11:00 IST 24 Nov).
- 2026-11-24 12:55 IST: EXP-2026-11-23-01 **completed** — ensemble MICRO 59.7 / MACRO 45.1 on the 136 lowres_poc test scenes (IoU 0.3); arty and spa weakest.
- 2026-11-24 19:01 IST: **v2 training running** — [EXP-2026-11-24-01](experiments/EXP-2026-11-24-01-lowres-poc-v2/record_ai.md): confusion-aware loss + radar/missile oversampling (y11l → y8m → TTA scoring; ETA ~08:00–10:00 IST 25 Nov). v1 analysis: the errors are class confusions (radar/missile/logistic/tank → vehicle), not missed detections.
- 2026-11-25 12:35 IST: **EXP-2026-11-24-01 completed** (03:48 IST). v2 ensemble MICRO 60.9 / MACRO 45.1 (v1 59.7 / 45.1); radar 52.2 → 58.9, missile 45.6 → 48.4; spa 26.0 → 8.3 and tank 42.1 → 38.6 regressed. Best single model y11l-v2 61.0 / 45.4. Per-class best-of all sets 63.0 / 49.4. [record](experiments/EXP-2026-11-24-01-lowres-poc-v2/record_ai.md)
