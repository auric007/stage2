# Train-subset loop — live status (AI-authored, written by code/brain.py after every decision)

Updated 2026-10-29 14:12 IST. Experiment record: [record_ai.md](record_ai.md). Stop the loop: `touch /sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/STOP`.

**Phase 2 / v3 (from ~19:00 IST):** the v2d search resolved at 71 images; influence re-computed on the two 71-removed models and candidates re-ranked margin-aware on object counts + positive influence (images whose help goes to classes near their PASS threshold — camo, smallvehicle, MICRO — are expensive; images helping classes with slack — tank, missile, arty — are cheap); paired seeds, restart at k = 16.

**Loop v2 (since 04:01 IST): equal optimizer steps.** Each subset trains 15 x (12,522 / its tiles) epochs, so every run gets the same steps and lr schedule. From step 7 (14:45 IST) each proposal is trained with seeds 0 and 1 in parallel and judged on the 2-seed mean; the search stops below 8-image steps. From step 5 every accepted level is re-trained with seed 1 and kept only if the 2-seed mean passes (else reverted). From step 2 (05:55 IST) proposals are class-balanced: a cluster is skipped if it would push any class's moved share above the moved image share + 3 pts. v1 (fixed 15 epochs) confounded removal with less training: [loop_status_v1_epochmatched_ai.md](loop_status_v1_epochmatched_ai.md).

**Baseline (full Train359, mean of 2 seeds):** MICRO 57.9 · MACRO 45.8 · noise MICRO 0.7 / MACRO 0.5

| class | arty | camo | logistic | missile | radar | smallvehicle | tank | vehicle |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| base F1 | 24.7 | 51.5 | 18.3 | 45.1 | 46.1 | 55.9 | 55.3 | 69.5 |
| seed noise | 4.3 | 2.7 | 1.6 | 0.7 | 0.9 | 1.3 | 1.6 | 0.9 |

**Current best (largest passing removal):** 71 of 359 Train images moved (63 clusters), Train keeps 288 images. Phase: idle. KEEP-marked clusters: 28. Next chunk k = 4 images.

Moved objects (of Train total): arty 119/1177 · camo 452/2742 · logistic 356/1915 · missile 78/711 · radar 53/421 · smallvehicle 1218/5349 · tank 122/1802 · vehicle 2798/12287

Lists: `/sfs/stage2/artifacts/EXP-2026-10-28-01-train-subset-influence-loop/state/best_removed_scenes.txt`, `best_keep_scenes.txt`. Influence files used: s03_A_rm71/tile_influence_epoch6_epoch12_last.json, s05_confirm_s03_A_rm71/tile_influence_epoch6_epoch12_last.json.

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
| s06_A_rm106 | 106 | 57.1 | 45.3 | 45.3 | 46.0 | 22.2 | FAIL | MICRO, smallvehicle |
| s06_B_rm88 | 88 | 55.1 | 44.0 | 44.3 | 44.7 | 22.2 | FAIL | MICRO, MACRO, smallvehicle, tank, vehicle |
| s07_A_rm88 | 88 | 54.6 | 45.3 | 45.1 | 48.4 | 22.8 | FAIL | MICRO, MACRO, vehicle |
| s07_A2_rm88 | 88 | 56.6 | 45.6 | 44.9 | 46.7 | 25.0 | FAIL | MICRO, vehicle |
| s08_A_rm79 | 79 | 55.3 | 44.2 | 44.4 | 45.8 | 17.0 | FAIL | MICRO, MACRO, arty, camo, smallvehicle |
| s08_A2_rm79 | 79 | 57.0 | 45.7 | 47.0 | 46.7 | 22.0 | FAIL | MICRO, camo |
| s10_A_rm87 | 87 | 55.8 | 44.9 | 42.5 | 45.9 | 18.1 | FAIL | MICRO, MACRO, arty, camo, radar, smallvehicle |
| s10_A2_rm87 | 87 | 56.2 | 44.2 | 43.4 | 46.5 | 18.8 | FAIL | MICRO, MACRO, arty, camo, radar, smallvehicle |
| s11_A_rm79 | 79 | 57.2 | 45.9 | 44.3 | 46.2 | 20.7 | FAIL | smallvehicle |
| s11_A2_rm79 | 79 | 58.0 | 46.2 | 44.9 | 46.6 | 26.3 | FAIL | smallvehicle |

## Decision log

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
- 2026-10-28 14:37 IST — s06_A_rm106 (A, 106 images removed): FAIL MICRO,smallvehicle MICRO 57.1 MACRO 45.3
- 2026-10-28 14:37 IST — s06_B_rm88 (B, 88 images removed): FAIL MICRO,MACRO,smallvehicle,tank,vehicle MICRO 55.1 MACRO 44.0
- 2026-10-28 14:37 IST — both fail (smallvehicle,tank,vehicle): mark 1 clusters KEEP ['C018']
- 2026-10-28 14:37 IST — k -> 17
- 2026-10-28 14:37 IST — step 7: proposals A s07_A_rm88 (+15 clusters); B s07_B_rm79 (+6 clusters)
- 2026-10-28 14:39 IST — (agent) step-7 single-seed proposals (A 88 / B 79) killed: at this resolution single-run noise (~±3 MICRO; 88 removed scored below 106) exceeds the effect. From step 7 each proposal runs seeds 0+1 in parallel, judged on the 2-seed mean; search stops when k < 8 images
- 2026-10-28 14:39 IST — brain resumed
- 2026-10-28 14:39 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 14:39 IST — influence ranking (re)loaded from ['tile_influence_epoch5_epoch10.json', 'tile_influence_last.json']
- 2026-10-28 14:39 IST — step 7: proposals A s07_A_rm88 (+15 clusters); A2 s07_A2_rm88 (+15 clusters)
- 2026-10-28 16:27 IST — s07_A_rm88 (A, 88 images removed): FAIL MICRO,MACRO,vehicle MICRO 54.6 MACRO 45.3
- 2026-10-28 16:27 IST — s07_A2_rm88 (A2, 88 images removed): FAIL MICRO,vehicle MICRO 56.6 MACRO 45.6
- 2026-10-28 16:27 IST — paired s07_A_rm88 + s07_A2_rm88 (88 images): 2-seed mean MICRO 55.6 MACRO 45.4 FAIL MICRO,vehicle
- 2026-10-28 16:27 IST — both fail (vehicle): mark 1 clusters KEEP ['C085']
- 2026-10-28 16:27 IST — k -> 8
- 2026-10-28 16:27 IST — step 8: proposals A s08_A_rm79 (+6 clusters); A2 s08_A2_rm79 (+6 clusters)
- 2026-10-28 18:13 IST — s08_A_rm79 (A, 79 images removed): FAIL MICRO,MACRO,arty,camo,smallvehicle MICRO 55.3 MACRO 44.2
- 2026-10-28 18:13 IST — s08_A2_rm79 (A2, 79 images removed): FAIL MICRO,camo MICRO 57.0 MACRO 45.7
- 2026-10-28 18:13 IST — paired s08_A_rm79 + s08_A2_rm79 (79 images): 2-seed mean MICRO 56.1 MACRO 44.9 FAIL MICRO,MACRO,arty,camo,smallvehicle
- 2026-10-28 18:13 IST — both fail (arty,camo,smallvehicle): mark 6 clusters KEEP ['C169', 'C259', 'C009', 'C265', 'C237', 'C045']
- 2026-10-28 18:13 IST — k -> 4
- 2026-10-28 18:13 IST — search resolved: k = 4 < 8 images is below what one step can measure; best = 71 images moved
- 2026-10-28 18:19 IST — PHASE 2 (v3, agent): v2d search resolved at 71. Influence re-computed with TracIn on the two 71-removed models (s03_A_rm71, s05_confirm; ckpts epoch6/epoch12/last). Candidates ranked margin-aware: per class, mean(object share, positive-influence share) / slack vs the PASS threshold at the 71 level (camo, smallvehicle, MICRO slack ~0.5 pt -> expensive; tank, missile, arty slack 5-6 pt -> cheap). Reason: steps 6-8 removed clusters with strongly negative full-model influence and still failed (all contained C169: 218 camo). Paired seeds, restart k = 16, KEEP marks retained. Previous state: state/brain_state_v2d_resolved.json, code: code/brain.py.bak_v2d.
- 2026-10-28 18:19 IST — brain resumed
- 2026-10-28 18:19 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-28 18:27 IST — influence ranking (re)loaded from ['s03_A_rm71/tile_influence_epoch6_epoch12_last.json', 's05_confirm_s03_A_rm71/tile_influence_epoch6_epoch12_last.json']
- 2026-10-28 18:27 IST — step 10: proposals A s10_A_rm87 (+16 clusters); A2 s10_A2_rm87 (+16 clusters); A adds objects arty 8, camo 1, logistic 12, missile 0, radar 5, smallvehicle 23, tank 18, vehicle 85
- 2026-10-28 19:11 IST — STOP file found — brain exits (running jobs continue to completion)
- 2026-10-29 11:35 IST — brain resumed
- 2026-10-29 11:35 IST — baseline ready: MICRO 57.9 MACRO 45.8 noise 0.7/0.5
- 2026-10-29 11:35 IST — influence ranking (re)loaded from ['s03_A_rm71/tile_influence_epoch6_epoch12_last.json', 's05_confirm_s03_A_rm71/tile_influence_epoch6_epoch12_last.json']
- 2026-10-29 11:35 IST — s10_A_rm87 (A, 87 images removed): FAIL MICRO,MACRO,arty,camo,radar,smallvehicle MICRO 55.8 MACRO 44.9
- 2026-10-29 11:35 IST — s10_A2_rm87 (A2, 87 images removed): FAIL MICRO,MACRO,arty,camo,radar,smallvehicle MICRO 56.2 MACRO 44.2
- 2026-10-29 11:35 IST — paired s10_A_rm87 + s10_A2_rm87 (87 images): 2-seed mean MICRO 56.0 MACRO 44.6 FAIL MICRO,MACRO,arty,camo,radar,smallvehicle
- 2026-10-29 11:35 IST — both fail (arty,camo,radar,smallvehicle): mark 1 clusters KEEP ['C116']
- 2026-10-29 11:35 IST — k -> 8
- 2026-10-29 11:35 IST — step 11: proposals A s11_A_rm79 (+8 clusters); A2 s11_A2_rm79 (+8 clusters); A adds objects arty 0, camo 0, logistic 1, missile 0, radar 3, smallvehicle 3, tank 13, vehicle 34
- 2026-10-29 13:22 IST — s11_A_rm79 (A, 79 images removed): FAIL smallvehicle MICRO 57.2 MACRO 45.9
- 2026-10-29 13:22 IST — s11_A2_rm79 (A2, 79 images removed): FAIL smallvehicle MICRO 58.0 MACRO 46.2
- 2026-10-29 13:22 IST — paired s11_A_rm79 + s11_A2_rm79 (79 images): 2-seed mean MICRO 57.6 MACRO 46.1 FAIL smallvehicle
- 2026-10-29 13:22 IST — both fail (smallvehicle): mark 1 clusters KEEP ['C235']
- 2026-10-29 13:22 IST — k -> 4
- 2026-10-29 13:22 IST — search resolved: k = 4 < 8 images is below what one step can measure; best = 71 images moved
