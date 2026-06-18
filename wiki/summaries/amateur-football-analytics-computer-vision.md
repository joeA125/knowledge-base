---
title: "Amateur Football Analytics Using Computer Vision — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, camera-calibration, object-detection, multi-object-tracking]
sources: [raw/papers/amateur_footbal_analytics_computer_vision.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Amateur Football Analytics Using Computer Vision

**Author:** Panagiotis Mavrogiannis
**Affiliation:** University of Piraeus, MSc Artificial Intelligence
**Published:** 2021 (Master's thesis)

## Key Contribution

Develops an end-to-end prototype application for extracting football analytics (player positions on a minimap, team detection, ball possession, attacking/defending stance) from single-camera broadcast footage, targeting low-budget amateur clubs. Combines existing methods — dual-GAN court detection, FootAndBall detector, Siamese-based [[camera-calibration]], and ECC camera tracking — into a unified pipeline running at 2.5–3 FPS on consumer-grade hardware (laptop with GTX 1060).

## Pipeline (Four Modules)

### 1. Court Detector
Dual conditional GAN (Pix2Pix) architecture from [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]]: a segmentation GAN separates grass from background, then a detection GAN extracts field markings (edge map) from the segmented foreground. Grass mask refinement removes spurious patches via contour analysis + Gaussian blur. Runs at ~15.5 FPS.

### 2. Camera Estimator
Based on the [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] Siamese retrieval + synthetic database approach (90K camera poses, 16-d features, FLANN nearest-neighbour search). Initial pose from retrieval is refined via Enhanced Correlation Coefficient Maximization (ECC). For subsequent frames, ECC tracks camera pose updates directly (50 iterations, $\epsilon = 10^{-3}$), avoiding expensive database search. Homography history with weighted averaging stabilises output.

### 3. Team Detector
- **Detection:** FootAndBall CNN (Komorowski et al., 2019) — lightweight single-stage detector with only 198K parameters, using Feature Pyramid Network design. Detects players and ball from masked frames.
- **Team classification:** Three methods tested. Best: HSV colour histograms (36 + 32 + 32 bins) clustered via K-means into 3 groups (team A, team B, referee). Trained on first 50 frames. Achieves perfect completeness on Belgium–Japan and Manchester–Chelsea test sets.
- **Tracking:** Pyramidal Lucas-Kanade optical flow tracks bounding box midpoints between detection frames (every 5 frames). Faster than correlation tracking (~0.02s vs ~0.07s per frame for 15 players).

### 4. Game Analytics
Perspective transformation maps bounding boxes to minimap coordinates. Logistic regression on first-frame top-view positions determines which goal belongs to which team. Ball possession estimated by nearest-player Euclidean distance; attacking/defending inferred from ball-holder's pitch half.

## Results

### Team Classification (Completeness Score)
| Method | AEK | Belgium | ISSIA | Man Utd | Egaleo | Time |
|---|---|---|---|---|---|---|
| Dominant Colour | 0.40 | 0.68 | 0.34 | 0.09 | — | 85.7s |
| **Colour Histogram** | **0.72** | **1.00** | **0.59** | **1.00** | **0.40** | **17.6s** |
| Segmentation | 0.72 | 1.00 | 0.59 | 0.45 | 0.27 | 25.1s |

### Key Limitations
- Camera estimation fails on zoomed midfield views with few visible markings.
- Ball detection has low recall (inherited from FootAndBall's limited ball training data).
- Goalkeepers classified randomly (different kit colour not handled).
- Player positions on minimap are jittery due to camera estimation instability.
- Green-kit teams cause colour suppression failures.

## Relation to Other Vault Papers

This thesis is an applied systems paper that integrates multiple methods already in the vault: the [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] camera pose engine and Siamese retrieval, and the broader [[camera-calibration]] and [[game-state-reconstruction]] paradigms. It demonstrates the accessibility challenges highlighted in [[detection-tracking-football-broadcast-footage|Tshiani (2025)]] and the CV football review [[computer-vision-football-review|Zheng et al. (2025)]].

## See Also

- [[camera-calibration]]
- [[game-state-reconstruction]]
- [[sports-camera-calibration-synthetic-data|Chen & Little Source Summary]]
- [[detection-tracking-football-broadcast-footage|Tshiani Source Summary]]
