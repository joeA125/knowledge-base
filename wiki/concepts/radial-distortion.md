---
title: "Radial Distortion"
type: concept
tags: [computer-vision, camera-calibration, projective-geometry, radial-distortion]
sources: [raw/papers/camera-calibration-benchmarking.md, raw/papers/tvcalib_camera_calibration_football.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-07
updated: 2026-07-07
---

# Radial Distortion

Radial distortion is a lens aberration that causes straight lines in the 3D world to appear curved in the image. It is characterised by displacement of image points radially from the optical centre, modelled by polynomial coefficients $k_1, k_2, \ldots$:

$$x_d = x_u(1 + k_1 r^2 + k_2 r^4 + \ldots)$$

where $x_u$ is the undistorted point, $x_d$ is the distorted observation, and $r$ is the radial distance from the principal point. Positive $k_1$ produces barrel distortion (common in wide-angle lenses); negative $k_1$ produces pincushion distortion.

## Why It Matters for Sports Broadcast

The [[camera-calibration-benchmarking|ProCC paper (Magera et al., 2025)]] demonstrates that radial distortion is not optional for broadcast camera calibration:

- On WC14, pinhole + one radial coefficient achieves JaC₅ **92.5** vs **79.1** for the best [[homography]] annotations — a 13.4-point gap attributable entirely to distortion modelling.
- On SoccerNet (21K images, more diverse cameras), adding distortion improves JaC₂ from 40.2 to **54.3** (+14.1 points).
- Disagreement between homography and pinhole+distortion models exceeds **2.5 metres** in some pitch regions, failing FIFA's 50cm standard for offside technology.

## Relation to Homographies

A [[homography]] is a planar projective transformation that inherently cannot model radial distortion. When broadcast cameras exhibit significant barrel distortion (which most do, especially wide-angle cameras), homography-based approaches produce systematically curved errors that no amount of improved annotation can fix. This is the core argument of the [[camera-calibration-benchmarking|ProCC paper]]: the problem is the camera model, not the annotation quality.

## Use in Other Vault Papers

- [[tvcalib-camera-calibration-football|TVCalib (Theiner & Ewerth, 2023)]]: Joint optimisation of $k_1, k_2$ improved AC@5 on WC14 by +4.0%, but introduced local minima at low FoV on SN-Calib. TVCalib treats distortion as optional.
- [[soccernet-game-state-reconstruction-improvement|Constructor Tech pipeline (Golovkin et al., 2024)]]: SegFormer regression predicts camera parameters but does not explicitly model distortion, which may limit precision for professional applications.

## See Also

- [[camera-calibration]]
- [[homography]]
- [[camera-calibration-benchmarking|ProCC Source Summary]]
- [[tvcalib-camera-calibration-football|TVCalib Source Summary]]
