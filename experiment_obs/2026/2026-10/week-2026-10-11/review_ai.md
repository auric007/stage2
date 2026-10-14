# Week of 2026-10-11 - Review — AI-authored

Author: AI agent (Claude). Human/legacy companion: [review.md](review.md).
Moved here on 2026-10-14 from `review.md` under the human/AI authorship rule (content unchanged except links).

## Update — 2026-10-14 01:40 IST (Claude)

Recorded research work this week: [EXP-2026-10-12-01](experiments/EXP-2026-10-12-01-attr-pattern-merge/record_ai.md)
(running, Oct 12–14): attribute × prediction pattern study → user rule set P03. Missile row locked
(R 67.1 % / P 72.1 %); radar row analysed (R 56.7 % / P 50.0 %), decisions pending. On Oct 13 the stage2 research home was created
and all earlier work imported; on Oct 14 tracking moved into this notebook ([daily](daily/2026-10-14_ai.md)).

## Overnight GPU queue — 2026-10-14 02:13–08:25 IST (Claude)

All 9 jobs completed, no failures (V100; [daily log](daily/2026-10-14_ai.md)). Missile P03 = researcher's rule set; F1 in %.

| run | form | deploy P01 F1 | recall-first micro / macro | missile P03 R / P (F1) | radar F1 |
|---|---|--:|--:|--:|--:|
| [05 baseline](experiments/EXP-2026-10-14-05-baseline-y8m-single/record_ai.md) | single y8m | 57.1 | 71.6 / 53.9 | 54.2 / 78.2 (64.0) | 50.6 |
| [04 control](experiments/EXP-2026-10-14-04-control-finetune-a-unchanged/record_ai.md) | single, +10 ep | 55.0 | 70.4 / 52.7 | 66.1 / 65.1 (65.6) | 53.0 |
| [01 B](experiments/EXP-2026-10-14-01-relabel-b-nonlauncher-vehicle/record_ai.md) | single, +10 ep | 55.6 | 70.1 / 52.1 | 64.6 / 70.7 (67.5) | 53.1 |
| [02 C](experiments/EXP-2026-10-14-02-relabel-c-unloaded-vehicle/record_ai.md) | single, +10 ep | 55.0 | 69.9 / 52.1 | 64.9 / 72.5 (68.5) | 52.3 |
| [03 D](experiments/EXP-2026-10-14-03-relabel-d-both-vehicle/record_ai.md) | single, +10 ep | 55.7 | 70.5 / 53.3 | 60.5 / 77.2 (67.9) | 53.9 |
| [09 ens control](experiments/EXP-2026-10-14-09-ens-control-y8mft-y11l/record_ai.md) | y8m-ft + y11l | 56.0 | 73.4 / 52.8 | 69.0 / 61.6 (65.1) | 52.1 |
| [06 ens B](experiments/EXP-2026-10-14-06-ens-b-y8mft-y11l/record_ai.md) | y8m-B + y11l | 56.1 | 73.4 / 52.7 | 68.3 / 65.8 (67.1) | 52.5 |
| [07 ens C](experiments/EXP-2026-10-14-07-ens-c-y8mft-y11l/record_ai.md) | y8m-C + y11l | 55.8 | 73.4 / 52.6 | 68.0 / 65.5 (66.7) | 51.2 |
| [08 ens D](experiments/EXP-2026-10-14-08-ens-d-y8mft-y11l/record_ai.md) | y8m-D + y11l | **56.3** | **75.0 / 58.5** | 68.7 / 69.1 (**68.9**) | 52.8 |
| m0001 (Sept pipeline) | y8m + y11l | 56.6 | 74.4 / 58.0 | 67.1 / 72.1 (69.5) | 53.2 |

Readings: (1) extra fine-tuning alone raises missile recall but costs precision and ~2 points of overall F1; (2) every relabel beats its
control on missile F1 (single +1.9…+2.9, ensemble +1.6…+3.8) — D strongest, mostly via precision; (3) in deployable form, ensemble D is
level with m0001 overall (recall-first slightly higher, missile F1 −0.6) — a like-for-like m0001 run through the same pipeline is running
(EXP-2026-10-14-10). Failure: the GPU sat idle 08:25–15:55 because the agent's wake-up watcher was not re-armed after D.
