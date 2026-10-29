# This week (25–31 October) — what we did, what still fails, what comes next

Author: AI agent (Claude), 2026-10-29. Human companion: [weekly_summary_plain.md](weekly_summary_plain.md).
Plain-language version of [weekly_detailed_report_ai.md](weekly_detailed_report_ai.md), which has the full figures and sources.

## The system in one paragraph

We are building software that looks at satellite images and points out military equipment: radars, missile launchers,
artillery, tanks, logistics trucks, ordinary vehicles, small cars and camouflaged objects. It has two parts:

- **The detector.** It scans the whole image and marks every object with its type. This is the main product.
- **The second-opinion AI** (a vision-language model). It looks closely at objects the detector is unsure about, mainly radars and missiles,
  and says whether they are really what the detector thinks.

---

## What we did this week

1. **Retrained the detector on the new, larger image collection** delivered on 27 October.
   It improved a little, mostly on artillery, camouflaged objects and tanks.
2. **Settled how to count camouflaged radars and missiles.** Counting a hidden missile as "missile", not as "camouflage",
   matches what the detector actually sees and gives clearly better results. This is now the rule.
3. **Chose the version to deploy.** For each equipment type we picked whichever of our trained models handles it best.
   We favoured finding more radars and missiles, even at the cost of a few more false alarms.
4. **Showed we can test on more images without losing quality.** About 1 in 5 training images (71 of 359) can be moved into the test
   collection and the detector stays just as good. A bigger test collection gives a more trustworthy picture.
5. **Compared the detector with the second-opinion AI** on the same radars and missiles.
   They often agree. But each one also catches some targets the other misses.
6. **Looked closely at every mistake** the deployed detector makes, to see where and why it fails (below).

---

## Where we stand today

| equipment | how many it finds | how often an alert is right |
|---|---|---|
| tanks | about 7 in 10 | about 8 in 10 |
| ordinary vehicles | about 7 in 10 | about 3 in 4 |
| radars | about 2 in 3 | about half |
| small cars | about 6 in 10 | about 6 in 10 |
| missile launchers | a bit over half | about half |
| camouflaged objects | about half | about 6 in 10 |
| artillery | about half | about half |
| logistics trucks | about 1 in 4 | about 1 in 3 |

The second-opinion AI is **not ready to work on its own**. It finds about 1 in 3 radars and about half of the missiles.
Only about 1 in 4 of its alerts is correct.

---

## What still fails, and why

### 1. Missile launchers are confused with ordinary trucks — the biggest problem

- About half of the missiles the detector misses are labelled "vehicle" instead.
- About 6 in 10 of its false missile alarms are actually ordinary trucks.
- It recognises classic air-defence launchers with their missiles loaded very well.
- It struggles with the launchers that really do look like trucks:
  - **rocket launchers (MLRS):** only about 1 in 4 found;
  - **launchers with no missiles on them,** or where nobody recorded whether they were loaded: only about 1 in 8 found;
  - **missile transport and reload trucks:** only about 1 in 5 found.
- **Where we failed:** last week we tried teaching the detector to call these truck-like launchers "vehicle". It looked promising on
  the old test images, but on the new, larger test collection it made no real difference. The simple fix did not work.

### 2. Rare equipment types are almost never found

- Some types were **never found once**: the K9 self-propelled gun and the HQ6 missile system.
- Others are found only about **1 time in 5**: self-propelled artillery, electronic-warfare radars, and transport vehicles under camouflage nets.
- The common types in the same groups are found almost every time: the usual radar models, PCL artillery and standard tanks.
- **The reason:** the detector has seen too few examples of the rare types during training.

### 3. Small and faint objects are missed, and empty ground raises false alarms

- Small cars and ordinary vehicles are often missed completely, especially the smallest ones.
- Many false alarms land on spots where nothing is marked.
- **Not yet checked:** some of these "false alarms" may be real vehicles that were never marked in the images. If so, the detector is
  better than the score says and the image labels need fixing.

### 4. Radars that are folded down or partly hidden

- Radars standing open are found about 3 times in 4.
- Folded-down radars, or ones whose position was not recorded, are found only about half the time or less.
- Smaller radars are missed more often.
- About half of all radar false alarms are on empty ground.

### 5. Logistics trucks versus ordinary vehicles

- Logistics is the weakest type: only about 1 in 4 is found.
- The detector often mixes logistics trucks up with ordinary vehicles, in both directions.
- The line between the two types may not be sharp in the labels either.

### 6. The newer test sites are harder

- On the newer half of the test images, the detector finds noticeably fewer missiles and tanks than on the older half.

### 7. The second-opinion AI receives the wrong mix of objects

- **Too much junk.** More than 9 out of 10 objects it is shown are not radars or missiles. It throws away most of them, but the
  junk that remains still outnumbers the real targets.
- **Some targets never reach it.** The step that hands objects over is set too strict: about 1 in 3 radars and 1 in 6 missiles
  are never shown to it.
- **When it is wrong,** it usually says "something else" rather than mixing radar up with missile.
- **Lower-quality images.** Some new test images were only enlarged in a simple way, not with the sharper enhancement it was trained on.

### 8. Running cost

- The deployed version runs several models on every image to get the best result for each equipment type, which makes it slow and heavy.

---

## What we should do next

**First, quick checks that need no heavy computing (this week):**

1. **Count the rare types in our training images.** For K9, HQ6, rocket launchers, self-propelled artillery and electronic-warfare radars,
   this tells us whether we need more images or just need to show the existing ones more often.
2. **Look at a sample of false alarms and misses by eye,** especially small cars and radars. This tells us how many are real detector
   errors and how many are missing labels. Fixing labels is a free improvement.
3. **Agree how truck-like missile launchers should be scored.** Should an empty launcher or a rocket launcher that the detector calls
   "vehicle" count as acceptable? The answer should reflect what the analysts actually need.
4. **Test the second-opinion AI on the same new test images as the detector,** so the two can finally be compared directly.

**Then, training work:**

5. **Train the detector with extra emphasis on rare types,** by repeating the images that contain them and pasting extra copies into others.
6. **Repeat the best training once more** to learn how much results change by chance. Then we know which improvements are real.
7. **Give the second-opinion AI better input:** either the detector's own boxes, or a looser hand-over step, so fewer real targets are lost.
8. **Retrain the second-opinion AI on the detector's own typical mistakes,** such as trucks called missiles and empty ground called radar,
   taken only from training images.

**Finally, put the two together:**

9. **Use the second-opinion AI only as a checker** of the detector's doubtful radar and missile alerts. It should confirm, reject or
   correct them. This aims straight at the biggest problem, the missile–truck mix-up. On the old test images, the two together could find
   about 6 in 10 radars and missiles, against about 5 in 10 for the detector alone.
10. **Build a lighter version of the detector** and measure how much accuracy the heavy version really buys, so a deployment choice can be made.
