# Train-subset loop v1 (fixed 15 epochs, STOPPED 04:10 IST: confounded fewer images with fewer steps) — AI-authored snapshot

Updated 2026-10-28 03:58 IST. Experiment record: [record_ai.md](record_ai.md). Stop the loop: `touch /sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/STOP`.

**Baseline (full Train359, mean of 2 seeds):** MICRO 57.9 · MACRO 45.8 · noise MICRO 0.7 / MACRO 0.5

| class | arty | camo | logistic | missile | radar | smallvehicle | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| base F1 | 24.7 | 51.5 | 18.3 | 45.1 | 46.1 | 55.9 | 55.3 | 69.5 |
| seed noise | 4.3 | 2.7 | 1.6 | 0.7 | 0.9 | 1.3 | 1.6 | 0.9 |

**Current best (largest passing removal):** 0 of 359 Train images moved (0 clusters), Train keeps 359 images. Phase: search. KEEP-marked clusters: 6. Next chunk k = 71 images.

Moved objects (of Train total): arty 0/1177 · camo 0/2742 · logistic 0/1915 · missile 0/711 · radar 0/421 · smallvehicle 0/5349 · tank 0/1802 · vehicle 0/12287

Lists: `/sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/state/best_removed_scenes.txt`, `best_keep_scenes.txt`. Influence files used: tile_influence_epoch5_epoch10.json, tile_influence_last.json.

| run | removed images | MICRO | MACRO | radar | missile | arty | verdict | failing |
|---|--:|--:|--:|--:|--:|--:|---|---|
| s01_A_rm143 | 143 | 49.6 | 39.1 | 39.0 | 39.5 | 17.0 | FAIL | MICRO, MACRO, arty, camo, missile, radar, smallvehicle, tank, vehicle |
| s01_B_rm71 | 71 | 55.0 | 43.1 | 38.9 | 45.5 | 21.1 | FAIL | MICRO, MACRO, radar, smallvehicle |

## Decision log

- 2026-10-28 01:05 IST — brain started
- 2026-10-28 01:06 IST — brain started
- 2026-10-28 02:41 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 02:43 IST — influence ranking (re)loaded from ['tile_influence_last.json']
- 2026-10-28 02:43 IST — step 1: proposals A s01_A_rm143 (+121 clusters); B s01_B_rm71 (+60 clusters)
- 2026-10-28 03:00 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 03:58 IST — s01_A_rm143 (A, 143 images removed): FAIL MICRO,MACRO,arty,camo,missile,radar,smallvehicle,tank,vehicle MICRO 49.6 MACRO 39.1
- 2026-10-28 03:58 IST — s01_B_rm71 (B, 71 images removed): FAIL MICRO,MACRO,radar,smallvehicle MICRO 55.0 MACRO 43.1
- 2026-10-28 03:58 IST — both fail (radar,smallvehicle): mark 6 clusters KEEP ['C015', 'C280', 'C294', 'C110', 'C123', 'C011']
- 2026-10-28 03:58 IST — k -> 71
- 2026-10-28 03:58 IST — step 2: proposals A s02_A_rm71 (+62 clusters); B s02_B_rm38 (+33 clusters)
