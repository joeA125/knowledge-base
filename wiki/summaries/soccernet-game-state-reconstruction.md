---
title: "SoccerNet Game State Reconstruction — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, camera-calibration, object-detection]
sources: [raw/papers/soccernet-game-state-reconstruction.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-06-18
---

# SoccerNet Game State Reconstruction

**Authors:** Vladimir Somers, Victor Joos, Anthony Cioppa, Silvio Giancola, et al.
**Affiliations:** UCLouvain, University of Liège, EPFL, KAUST, Sportradar
**Published:** 2024 (CVPR 2024)

## Key Contribution

Introduces [[game-state-reconstruction]] (GSR), a novel computer vision task for football analytics that combines tracking, identification (role, team, jersey number), and pitch localisation into a unified benchmark. Releases SoccerNet-GSR (200 annotated 30-second clips, 9.37M line points, 2.36M athlete annotations), the GS-HOTA evaluation metric, and an open-source baseline pipeline.

## Task Definition

Given a broadcast video, predict for each frame and each athlete: pitch coordinates ($x$, $y$), role (player/goalkeeper/referee/other), team (left/right), and jersey number. Output is a minimap visualisation of the game state.

## GS-HOTA Metric

Extends HOTA with: (1) LocSim — Gaussian kernel on Euclidean pitch distance ($\tau = 5$m tolerance); (2) IdSim — binary match requiring all attributes correct. This strict constraint means a single wrong attribute turns a detection into a false positive.

## Baseline Pipeline (GSR-Baseline)

YOLOv8 detection → StrongSORT tracking → PRTreID for role/team-aware embeddings → [[tvcalib-camera-calibration-football|TVCalib]] for pitch localisation and [[camera-calibration]] → MMOCR for jersey number recognition → majority voting for tracklet consistency → K-means for team affiliation.

The pipeline relies on [[homography]] estimation to map image-space detections to pitch coordinates, making camera calibration the critical link between detection and localisation.

## Results

Full GS-HOTA on test: **22.26%**. Jersey number recognition is the biggest bottleneck (drops score from 42.65% to 25.65% when enabled). Camera calibration failures cause catastrophic localisation errors.

## See Also

- [[game-state-reconstruction]]
- [[camera-calibration]]
- [[homography]]
- [[tvcalib-camera-calibration-football|TVCalib]]
- [[soccernet-game-state-reconstruction-improvement|SOTA Improvement]]
- [[amateur-football-analytics-computer-vision|Mavrogiannis Thesis]]
