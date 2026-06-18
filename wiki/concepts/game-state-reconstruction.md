---
title: "Game State Reconstruction"
type: concept
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, camera-calibration, object-detection]
sources: [raw/papers/soccernet-game-state-reconstruction.md, raw/papers/soccernet-game-state-reconstruction-improvement.md, raw/papers/soccernet-v2-action-spotting.md, raw/papers/detection-tracking-football-broadcast-footage.md, raw/papers/amateur_footbal_analytics_computer_vision.md]
confidence: 0.95
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-06-18
---

# Game State Reconstruction

Game State Reconstruction (GSR) is a computer vision task that aims to reconstruct the full state of a team sport from broadcast video: the 2D pitch positions, roles, team affiliations, and jersey numbers of all athletes, visualised on a minimap.

## Sub-Tasks

GSR decomposes into several interconnected sub-tasks:

1. **Object detection:** Locating athletes and the ball in each frame. Typically YOLO-family detectors (which use [[feature-pyramid-network]]s for multi-scale detection), or lightweight alternatives like FootAndBall (198K params).
2. **Multi-object tracking:** Maintaining consistent identities across frames (DeepSORT, ByteTrack, StrongSORT) or via [[optical-flow]] (pyramidal Lucas-Kanade).
3. **[[camera-calibration|Camera calibration]] / pitch localisation:** Estimating camera parameters to map image coordinates to real-world pitch coordinates via [[homography]] estimation. Approaches include [[siamese-network]] retrieval from synthetic databases ([[sports-camera-calibration-synthetic-data|Chen & Little, 2019]]), direct parameter optimisation ([[tvcalib-camera-calibration-football|TVCalib]]), or SegFormer regression. [[enhanced-correlation-coefficient|ECC maximization]] tracks camera pose between frames.
4. **Athlete identification:** Role classification (player/goalkeeper/referee), team affiliation (clustering ReID/TeamID embeddings), and jersey number recognition (OCR or classification heads).

## Court Detection

Field markings are detected via [[conditional-gan|dual conditional GANs (Pix2Pix)]] that segment grass from background and extract edge maps, providing the input for camera calibration.

## Evaluation: GS-HOTA

The GS-HOTA metric extends HOTA with a combined similarity score: LocSim (Gaussian kernel on Euclidean pitch distance, $\tau = 5$m) $\times$ IdSim (binary: all attributes must match). This strict design means any attribute error turns a detection into a false positive.

## State of the Art

The [[soccernet-game-state-reconstruction|original baseline]] achieved 22.26% GS-HOTA. The [[soccernet-game-state-reconstruction-improvement|Constructor Tech pipeline]] (2024 challenge winner) reached **63.81%** through improved camera parameter estimation, keypoint-based refinement, sophisticated post-processing, and a modular real-time architecture.

## Key Challenges

- **Camera calibration:** Failures when few pitch lines are visible cause catastrophic localisation errors.
- **Jersey number recognition:** The single biggest performance bottleneck; digits are often occluded or too small.
- **Ball detection:** Small size and rapid motion yield low recall even with high-resolution inputs.
- **Interdependencies:** Errors compound across sub-tasks (e.g., bad calibration renders perfect tracking useless).

## Accessibility

The [[amateur-football-analytics-computer-vision|Mavrogiannis thesis (2021)]] and [[detection-tracking-football-broadcast-footage|Tshiani (2025)]] demonstrate that meaningful analytics can be extracted on consumer hardware with single-camera setups, making GSR accessible to amateur clubs.

## See Also

- [[camera-calibration]]
- [[homography]]
- [[siamese-network]]
- [[optical-flow]]
- [[enhanced-correlation-coefficient]]
- [[conditional-gan]]
- [[feature-pyramid-network]]
- [[semantic-segmentation]]
