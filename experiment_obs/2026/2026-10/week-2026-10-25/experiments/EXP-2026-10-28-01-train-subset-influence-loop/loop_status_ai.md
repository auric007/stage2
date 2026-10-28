# Train-subset loop — live status (AI-authored, written by code/brain.py after every decision)

Updated 2026-10-28 12:50 IST. Experiment record: [record_ai.md](record_ai.md). Stop the loop: `touch /sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/STOP`.

**Loop v2 (since 04:01 IST): equal optimizer steps.** Each subset trains 15 x (12,522 / its tiles) epochs, so every run gets the same steps and lr schedule. From step 5 every accepted level is re-trained with seed 1 and kept only if the 2-seed mean passes (else reverted). From step 2 (05:55 IST) proposals are class-balanced: a cluster is skipped if it would push any class's moved share above the moved image share + 3 pts. v1 (fixed 15 epochs) confounded removal with less training: [loop_status_v1_epochmatched_ai.md](loop_status_v1_epochmatched_ai.md).

**Baseline (full Train359, mean of 2 seeds):** MICRO 57.9 · MACRO 45.8 · noise MICRO 0.7 / MACRO 0.5

| class | arty | camo | logistic | missile | radar | smallvehicle | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| base F1 | 24.7 | 51.5 | 18.3 | 45.1 | 46.1 | 55.9 | 55.3 | 69.5 |
| seed noise | 4.3 | 2.7 | 1.6 | 0.7 | 0.9 | 1.3 | 1.6 | 0.9 |

**Current best (largest passing removal):** 71 of 359 Train images moved (63 clusters), Train keeps 288 images. Phase: search. KEEP-marked clusters: 18. Next chunk k = 35 images.

Moved objects (of Train total): arty 119/1177 · camo 452/2742 · logistic 356/1915 · missile 78/711 · radar 53/421 · smallvehicle 1218/5349 · tank 122/1802 · vehicle 2798/12287

Lists: `/sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/state/best_removed_scenes.txt`, `best_keep_scenes.txt`. Influence files used: tile_influence_epoch5_epoch10.json, tile_influence_last.json.

| run | removed images | MICRO | MACRO | radar | missile | arty | verdict | failing |
|---|--:|--:|--:|--:|--:|--:|---|---|
| s01_A_rm143 | 143 | 54.6 | 45.6 | 43.1 | 47.1 | 31.2 | FAIL | MICRO, radar, smallvehicle, vehicle |
| s01_B_rm71 | 71 | 56.3 | 43.3 | 41.3 | 44.9 | 17.9 | FAIL | MICRO, MACRO, arty, camo, radar, smallvehicle |
| s02_A_rm143 | 143 | 54.5 | 43.2 | 41.2 | 42.8 | 21.4 | FAIL | MICRO, MACRO, missile, radar, smallvehicle, vehicle |
| s02_B_rm71 | 71 | 57.5 | 45.5 | 45.6 | 44.6 | 16.8 | FAIL | arty |
| s03_A_rm71 | 71 | 58.0 | 46.4 | 47.2 | 50.0 | 26.4 | PASS |  |
| s03_B_rm35 | 35 | 54.3 | 43.2 | 42.6 | 45.9 | 17.0 | FAIL | MICRO, MACRO, arty, radar, tank, vehicle |
| s04_random | 71 | 55.2 | 42.9 | 40.1 | 47.6 | 16.9 | FAIL | MICRO, MACRO, arty, radar, smallvehicle, tank |
| s04_A_rm214 | 214 | 52.4 | 42.9 | 45.5 | 49.7 | 31.9 | FAIL | MICRO, MACRO, camo, smallvehicle, tank, vehicle |
| s05_confirm_s03_A_rm71 | 71 | 57.4 | 45.9 | 44.4 | 48.0 | 25.4 | FAIL | camo |
| s05_A_rm142 | 142 | 55.2 | 45.3 | 44.8 | 48.8 | 29.3 | FAIL | MICRO, camo, smallvehicle, vehicle |

## Decision log

- 2026-10-28 04:01 IST — brain started
- 2026-10-28 04:01 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 04:01 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 04:01 IST — step 1: proposals A s01_A_rm143 (+124 clusters); B s01_B_rm71 (+61 clusters)
- 2026-10-28 04:01 IST — brain started
- 2026-10-28 04:01 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 04:01 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 05:46 IST — s01_A_rm143 (A, 143 images removed): FAIL MICRO,radar,smallvehicle,vehicle MICRO 54.6 MACRO 45.6
- 2026-10-28 05:46 IST — s01_B_rm71 (B, 71 images removed): FAIL MICRO,MACRO,arty,camo,radar,smallvehicle MICRO 56.3 MACRO 43.3
- 2026-10-28 05:46 IST — both fail (arty,camo,radar,smallvehicle): mark 2 clusters KEEP ['C011', 'C200']
- 2026-10-28 05:46 IST — k -> 71
- 2026-10-28 05:46 IST — step 2: proposals A s02_A_rm71 (+62 clusters); B s02_B_rm37 (+32 clusters)
- 2026-10-28 05:49 IST — (agent) step-2 proposals s02_A_rm71/s02_B_rm37 killed: unbalanced picks over-moved vehicle/smallvehicle (step 1 A: 40% images, 50-53% of those classes). Selection now class-balanced (cap = image share + 3 pts); k reset to 143
- 2026-10-28 05:49 IST — brain resumed
- 2026-10-28 05:49 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 05:49 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 05:49 IST — step 2: proposals A s02_A_rm143 (+126 clusters); B s02_B_rm71 (+63 clusters)
- 2026-10-28 07:34 IST — s02_A_rm143 (A, 143 images removed): FAIL MICRO,MACRO,missile,radar,smallvehicle,vehicle MICRO 54.5 MACRO 43.2
- 2026-10-28 07:34 IST — s02_B_rm71 (B, 71 images removed): FAIL arty MICRO 57.5 MACRO 45.5
- 2026-10-28 07:34 IST — both fail (arty): mark 2 clusters KEEP ['C067', 'C125']
- 2026-10-28 07:34 IST — k -> 71
- 2026-10-28 07:34 IST — step 3: proposals A s03_A_rm71 (+63 clusters); B s03_B_rm35 (+35 clusters)
- 2026-10-28 09:20 IST — s03_A_rm71 (A, 71 images removed): PASS MICRO 58.0 MACRO 46.4
- 2026-10-28 09:20 IST — s03_B_rm35 (B, 35 images removed): FAIL MICRO,MACRO,arty,radar,tank,vehicle MICRO 54.3 MACRO 43.2
- 2026-10-28 09:20 IST — accept A: +63 clusters -> 71 images moved; k -> 142
- 2026-10-28 09:20 IST — step 4: proposals random s04_random (+60 clusters); A s04_A_rm214 (+120 clusters)
- 2026-10-28 09:23 IST — (agent) single-run noise is ~±3 MICRO (s03_B, 35 removed: 54.3 vs s03_A, 71 removed: 58.0). From step 5 every accepted level is re-trained with seed 1 and kept only if the 2-seed mean passes; otherwise it is reverted
- 2026-10-28 09:23 IST — brain resumed
- 2026-10-28 09:23 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 09:23 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 11:06 IST — s04_random (random, 71 images removed): FAIL MICRO,MACRO,arty,radar,smallvehicle,tank MICRO 55.2 MACRO 42.9
- 2026-10-28 11:06 IST — s04_A_rm214 (A, 214 images removed): FAIL MICRO,MACRO,camo,smallvehicle,tank,vehicle MICRO 52.4 MACRO 42.9
- 2026-10-28 11:06 IST — both fail (camo,smallvehicle,tank,vehicle): mark 7 clusters KEEP ['C294', 'C189', 'C249', 'C181', 'C121', 'C193', 'C033']
- 2026-10-28 11:06 IST — k -> 71
- 2026-10-28 11:06 IST — step 5: proposals confirm s05_confirm_s03_A_rm71 (+0 clusters); A s05_A_rm142 (+63 clusters)
- 2026-10-28 12:50 IST — s05_confirm_s03_A_rm71 (confirm, 71 images removed): FAIL camo MICRO 57.4 MACRO 45.9
- 2026-10-28 12:50 IST — s05_A_rm142 (A, 142 images removed): FAIL MICRO,camo,smallvehicle,vehicle MICRO 55.2 MACRO 45.3
- 2026-10-28 12:50 IST — confirm level s03_A_rm71 (71 images) with seed 1: 2-seed mean MICRO 57.7 MACRO 46.2 -> CONFIRMED
- 2026-10-28 12:50 IST — both fail (camo,smallvehicle,vehicle): mark 7 clusters KEEP ['C239', 'C219', 'C205', 'C233', 'C137', 'C112', 'C077']
- 2026-10-28 12:50 IST — k -> 35
- 2026-10-28 12:50 IST — step 6: proposals A s06_A_rm106 (+31 clusters); B s06_B_rm88 (+14 clusters)
