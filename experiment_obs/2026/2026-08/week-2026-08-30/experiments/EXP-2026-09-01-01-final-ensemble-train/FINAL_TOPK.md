# Top-k class analysis — FINAL ensemble (per-box 8-class score vectors)

Each detection carries all 8 sigmoid class scores (multi-label ensemble, noisy-or fused over y8m+y11l × TTA). For every GT object we take the best-overlapping box (IoU ≥ gate, highest score) and check whether the GT's **true class** is that box's **top-1 (argmax), top-2, or top-3** score. `top-1` = ordinary correct classification; the top-2/top-3 lift shows recall recoverable if a box may propose its 2nd/3rd class. `missed` = no box overlapped the object.

## IoU 0.3

| class | GT | top-1 % | top-2 % | top-3 % | detected % | missed | top-1 wrong→ (dominant) |
|---|--:|--:|--:|--:|--:|--:|---|
| arty | 211 | 34.1 | 42.7 | 50.2 | 66.4 | 71 | tank (31) |
| camo | 523 | 52.6 | 63.9 | 78.8 | 91.0 | 47 | tank (103) |
| logistic | 241 | 29.0 | 63.1 | 72.2 | 85.5 | 35 | vehicle (121) |
| missile | 307 | 48.9 | 72.3 | 82.1 | 97.4 | 8 | vehicle (115) |
| radar | 206 | 42.7 | 62.1 | 72.3 | 91.3 | 18 | vehicle (71) |
| smallvehicle | 822 | 68.9 | 76.2 | 80.4 | 89.4 | 87 | vehicle (146) |
| tank | 737 | 68.7 | 86.0 | 97.0 | 98.6 | 10 | vehicle (160) |
| vehicle | 2053 | 89.3 | 95.1 | 96.0 | 97.0 | 61 | smallvehicle (61) |
| **MICRO** | 5100 | 69.8 | 81.2 | 87.1 | 93.4 | 337 | |
| **MACRO** | | 54.3 | 70.2 | 78.6 | | | |

## IoU 0.5

| class | GT | top-1 % | top-2 % | top-3 % | detected % | missed | top-1 wrong→ (dominant) |
|---|--:|--:|--:|--:|--:|--:|---|
| arty | 211 | 31.3 | 39.3 | 45.5 | 57.3 | 90 | tank (29) |
| camo | 523 | 50.5 | 61.6 | 76.3 | 87.8 | 64 | tank (101) |
| logistic | 241 | 28.6 | 56.8 | 65.6 | 77.2 | 55 | vehicle (103) |
| missile | 307 | 48.5 | 71.3 | 78.5 | 92.5 | 23 | vehicle (109) |
| radar | 206 | 36.4 | 52.4 | 62.1 | 77.2 | 47 | vehicle (64) |
| smallvehicle | 822 | 65.5 | 72.5 | 76.6 | 85.6 | 118 | vehicle (144) |
| tank | 737 | 66.9 | 84.3 | 95.1 | 96.6 | 25 | vehicle (159) |
| vehicle | 2053 | 88.2 | 93.8 | 94.5 | 95.5 | 92 | smallvehicle (58) |
| **MICRO** | 5100 | 67.9 | 78.6 | 84.2 | 89.9 | 514 | |
| **MACRO** | | 52.0 | 66.5 | 74.3 | | | |

## Reading it
- **top-1** ≈ ordinary recall (true class is the box's argmax).
- **top-2 / top-3** = the object is found and its true class is the 2nd/3rd-ranked score — i.e. recoverable by a light re-ranking / 2nd-stage classifier, NOT a detection miss.
- **top-1 wrong→** names the class that outranked the true one (usually `vehicle`), quantifying the confusion the vectors could fix.
