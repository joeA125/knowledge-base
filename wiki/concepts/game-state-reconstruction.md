---
title: "Game State Reconstruction"
type: concept
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, camera-calibration]
sources: [raw/papers/soccernet-game-state-reconstruction.md, raw/papers/soccernet-game-state-reconstruction-improvement.md, raw/papers/soccernet-v2-action-spotting.md, raw/papers/detection-tracking-football-broadcast-footage.md]
confidence: 0.95
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-05-26
---

# Game State Reconstruction

Game State Reconstruction (GSR) is a computer vision task that aims to reconstruct the full state of a team sport from broadcast video: the 2D pitch positions, roles, team affiliations, and jersey numbers of all athletes, visualised on a minimap.

## Sub-Tasks

GSR decomposes into several interconnected sub-tasks:

1. **Object detection:** Locating athletes and the ball in each frame (typically YOLO-family detectors).
2. **Multi-object tracking:** Maintaining consistent identities across frames (DeepSORT, ByteTrack, StrongSORT).
3. **Camera calibration / pitch localisation:** Estimating camera parameters to map image coordinates to real-world pitch coordinates (homography estimation, keypoint detection, SegFormer regression).
4. **Athlete identification:** Role classification (player/goalkeeper/referee), team affiliation (clustering ReID/TeamID embeddings), and jersey number recognition (OCR or classification heads).

## Evaluation: GS-HOTA

The GS-HOTA metric extends HOTA with a combined similarity score: LocSim (Gaussian kernel on Euclidean pitch distance, $\tau = 5$m) $\times$ IdSim (binary: all attributes must match). This strict design means any attribute error turns a detection into a false positive.

## State of the Art

The [[soccernet-game-state-reconstruction|original baseline]] achieved 22.26% GS-HOTA. The [[soccernet-game-state-reconstruction-improvement|Constructor Tech pipeline]] (2024 challenge winner) reached **63.81%** through improved camera parameter estimation, keypoint-based refinement, sophisticated post-processing, and a modular real-time architecture.

## Key Challenges

- **Camera calibration:** Failures when few pitch lines are visible cause catastrophic localisation errors.
- **Jersey number recognition:** The single biggest performance bottleneck; digits are often occluded or too small.
- **Ball detection:** Small size and rapid motion yield low recall even with high-resolution inputs.
- **Interdependencies:** Errors compound across sub-tasks (e.g., bad calibration renders perfect tracking useless).

## Applications

Coaching and tactical analysis, scouting, medical staff monitoring, fan engagement, augmented broadcast graphics, and automated highlight generation.

## See Also

- [[semantic-segmentation]]
- [[residual-connections]]
- [[batch-normalization]]
