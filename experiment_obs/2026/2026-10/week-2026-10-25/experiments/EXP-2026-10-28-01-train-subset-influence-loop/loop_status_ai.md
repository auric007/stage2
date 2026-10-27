# Train-subset loop — live status (AI-authored, written by code/brain.py after every decision)

Updated 2026-10-28 04:01 IST. Experiment record: [record_ai.md](record_ai.md). Stop the loop: `touch /sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/STOP`.

**Loop v2 (since 04:01 IST): equal optimizer steps.** Each subset trains 15 x (12,522 / its tiles) epochs, so every run gets the same steps and lr schedule. v1 (fixed 15 epochs) confounded removal with less training: [loop_status_v1_epochmatched_ai.md](loop_status_v1_epochmatched_ai.md).

**Baseline (full Train359, mean of 2 seeds):** MICRO 57.9 · MACRO 45.8 · noise MICRO 0.7 / MACRO 0.5

| class | arty | camo | logistic | missile | radar | smallvehicle | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| base F1 | 24.7 | 51.5 | 18.3 | 45.1 | 46.1 | 55.9 | 55.3 | 69.5 |
| seed noise | 4.3 | 2.7 | 1.6 | 0.7 | 0.9 | 1.3 | 1.6 | 0.9 |

**Current best (largest passing removal):** 0 of 359 Train images moved (0 clusters), Train keeps 359 images. Phase: search. KEEP-marked clusters: 0. Next chunk k = 143 images.

Moved objects (of Train total): arty 0/1177 · camo 0/2742 · logistic 0/1915 · missile 0/711 · radar 0/421 · smallvehicle 0/5349 · tank 0/1802 · vehicle 0/12287

Lists: `/sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/state/best_removed_scenes.txt`, `best_keep_scenes.txt`. Influence files used: none yet.

| run | removed images | MICRO | MACRO | radar | missile | arty | verdict | failing |
|---|--:|--:|--:|--:|--:|--:|---|---|

## Decision log

- 2026-10-28 04:01 IST — brain started
- 2026-10-28 04:01 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 04:01 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 04:01 IST — step 1: proposals A s01_A_rm143 (+124 clusters); B s01_B_rm71 (+61 clusters)
- 2026-10-28 04:01 IST — brain started
- 2026-10-28 04:01 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
