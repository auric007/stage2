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
- 2026-11-25 13:25 IST: **v3 training running** — [EXP-2026-11-25-01](experiments/EXP-2026-11-25-01-lowres-poc-v3/record_ai.md): 8 classes (tank_spa), tank_spa weight + background penalty, softer tank penalties; compared with v1/v2 in 8-class space (v2 ens: MICRO 61.0, MACRO(8) 48.5, tank_spa 29.2).
- 2026-11-25 19:12 IST: **y11l-v3 scored** (stopped on request at ep38, best ep26): MICRO 62.5 / MACRO(8) 49.8 — best single or ensemble so far (v2 ens 61.0 / 48.5); tank_spa 36.0 (v2 y11l 25.0, v1 ens 38.1), radar 57.7, missile 48.9. y8m-v3 + ensemble running (ETA ~01:00 IST 26 Nov).
- 2026-11-25 20:15 IST: v3 paused (y8m-v3 stopped on request at epoch 8). **Context-vs-object experiment running** — [EXP-2026-11-25-02](experiments/EXP-2026-11-25-02-context-vs-object/record_ai.md): phase 0 probe of y11l-v3 with context removed, phase 1 classifier ablation FULL / CONTEXT / SITE / OBJECT / SHAPE × 48/96/192 m.
- 2026-11-26 13:32 IST: **EXP-2026-11-25-02 completed** — object ≈ context (45 vs 44 macro-F1 clean), combined 48; missile/vehicle context classes, radar object class; 96 → 192 m patches no gain → larger tiles not supported. v3 paused.
