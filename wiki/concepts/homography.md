---
title: "Homography"
type: concept
tags: [computer-vision, projective-geometry, camera-calibration, sports-analytics]
sources: [raw/papers/tvcalib_camera_calibration_football.md, raw/papers/sports-camera_calibration-synthetic_data.md, raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/soccernet-game-state-reconstruction.md, raw/papers/soccernet-v2-action-spotting.md]
confidence: 0.95
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Homography

A homography is a $3 \times 3$ invertible matrix $H$ that maps points between two planes under perspective projection: $\mathbf{y'} = H\mathbf{y}$, where $\mathbf{y}$ and $\mathbf{y'}$ are homogeneous 2D coordinates. It has 8 degrees of freedom (9 entries minus 1 for scale).

## Role in Sports Field Registration

In sports broadcast CV, the field surface is approximately planar, so a homography can map between the image and a top-down field template. This is the dominant formulation for [[camera-calibration]]:

- **Direct Linear Transform (DLT):** Given $\geq 4$ point correspondences between the image and the field template, DLT solves for $H$ via SVD. Used as a baseline in [[tvcalib-camera-calibration-football|TVCalib]] and the [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]].
- **Synthetic retrieval:** [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] generate a database of synthetic camera poses with known homographies, then retrieve the nearest match via Siamese features.
- **Refinement:** Lucas-Kanade, Spatial Transformer Networks, or [[enhanced-correlation-coefficient|ECC]] align an initial homography estimate to the observed field markings.

## Relation to Camera Parameters

A homography is a degenerate case of the full camera projection: if all 3D points lie on a plane ($Z = 0$), then $H = K R^{[1,2]} [I | -t]$ where $K$ is the intrinsic matrix and $R, t$ are extrinsics. Individual camera parameters ($FoV$, pan, tilt, roll, position) can be recovered via **homography decomposition**, but this introduces additional errors — a key motivation for [[tvcalib-camera-calibration-football|TVCalib's]] direct parameter optimisation approach.

## Limitation: Planar Assumption

Homographies cannot model 3D elements that lie off the ground plane (goal posts, crossbars, airborne balls). [[tvcalib-camera-calibration-football|TVCalib]] showed that homography decomposition degrades accuracy specifically for goal segments, motivating a shift toward full 3D [[camera-calibration]].

## See Also

- [[camera-calibration]]
- [[game-state-reconstruction]]
