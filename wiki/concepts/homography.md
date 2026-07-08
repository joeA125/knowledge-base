---
title: "Homography"
type: concept
tags: [computer-vision, projective-geometry, camera-calibration, sports-analytics]
sources: [raw/papers/tvcalib_camera_calibration_football.md, raw/papers/sports-camera_calibration-synthetic_data.md, raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/soccernet-game-state-reconstruction.md, raw/papers/soccernet-v2-action-spotting.md, raw/papers/camera-calibration-benchmarking.md]
confidence: 0.95
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-07-07
---

# Homography

A homography is a $3 \times 3$ invertible matrix $H$ that maps points between two planes under perspective projection: $\mathbf{y'} = H\mathbf{y}$, where $\mathbf{y}$ and $\mathbf{y'}$ are homogeneous 2D coordinates. It has 8 degrees of freedom (9 entries minus 1 for scale).

## Role in Sports Field Registration

In sports broadcast CV, the field surface is approximately planar, so a homography can map between the image and a top-down field template. This is the dominant formulation for sports field registration (though [[camera-calibration-benchmarking|Magera et al. (2025)]] argue this is not true [[camera-calibration]]):

- **Direct Linear Transform (DLT):** Given $\geq 4$ point correspondences between the image and the field template, DLT solves for $H$ via SVD. Used as a baseline in [[tvcalib-camera-calibration-football|TVCalib]] and the [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]].
- **Synthetic retrieval:** [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] generate a database of synthetic camera poses with known homographies, then retrieve the nearest match via [[siamese-network|Siamese features]].
- **Refinement:** Lucas-Kanade, Spatial Transformer Networks, or [[enhanced-correlation-coefficient|ECC]] align an initial homography estimate to the observed field markings.

## Relation to Camera Parameters

A homography is a degenerate case of the full camera projection: if all 3D points lie on a plane ($Z = 0$), then $H = K R^{[1,2]} [I | -t]$ where $K$ is the intrinsic matrix and $R, t$ are extrinsics. Individual camera parameters ($FoV$, pan, tilt, roll, position) can be recovered via **homography decomposition**, but this introduces additional errors — a key motivation for [[tvcalib-camera-calibration-football|TVCalib's]] direct parameter optimisation approach.

## Fundamental Limitations

### Planar Assumption
Homographies cannot model 3D elements that lie off the ground plane (goal posts, crossbars, airborne balls). [[tvcalib-camera-calibration-football|TVCalib]] showed that homography decomposition degrades accuracy specifically for goal segments.

### Cannot Model Lens Distortion
[[camera-calibration-benchmarking|Magera et al. (2025)]] demonstrate that homographies inherently cannot capture [[radial-distortion]] from broadcast camera lenses. On WC14, the best homography annotations (CARWC) achieve JaC₅ of only **79.1**, while a pinhole model with one radial distortion coefficient reaches **92.5**. The disagreement between models exceeds **2.5 metres** in some pitch regions — failing FIFA's 50cm accuracy standard for offside technology. The problem is the camera model itself, not the annotation quality.

### Benchmark Bias
Most existing datasets (WC14, TS-WorldCup, CARWC, volleyball) provide only homography annotations as ground truth, systematically biasing the field toward registration methods and obscuring the limitations of the homography model. The [[jac-metric|JaC metric (ProCC)]] was designed to break this dependency.

## See Also

- [[camera-calibration]]
- [[radial-distortion]]
- [[jac-metric]]
- [[game-state-reconstruction]]
