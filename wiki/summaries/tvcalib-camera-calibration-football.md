---
title: "TVCalib: Camera Calibration for Sports Field Registration in Soccer — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, camera-calibration, semantic-segmentation]
sources: [raw/papers/tvcalib_camera_calibration_football.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-06-15
---

# TVCalib: Camera Calibration for Sports Field Registration in Soccer

**Authors:** Jonas Theiner, Ralph Ewerth
**Affiliations:** L3S Research Center / Leibniz University Hannover, TIB – Leibniz Information Centre for Science and Technology
**Published:** 2023 (CVPR Workshop)

## Key Contribution

Reframes sports field registration as a [[camera-calibration]] problem rather than homography estimation. Introduces a differentiable **segment reprojection loss** that learns individual camera parameters (position, rotation, FoV) from segment correspondences (lines, point clouds) by iterative gradient-based optimisation — without requiring keypoint correspondences or training data for the calibration module. The method estimates camera pose and focal length in a single step, unlike prior two-stage approaches (initial estimation + refinement).

## Method

### Calibration Object & Camera Model
The soccer field is modelled as a set of labelled segments: line segments (parametric form) and point cloud segments (circle arcs). A standard pinhole camera $P = KR[I | -t]$ is parameterised by $\phi = (FoV, t, pan, tilt, roll)$, with FoV predicted in Normalised Device Coordinates for numerical stability.

### Segment Reprojection Loss
For each visible segment, the loss measures the mean Euclidean distance between annotated/predicted pixels and the reprojected 3D segment:
- **Point–line distance:** Perpendicular distance from pixel to reprojected line.
- **Point–point-cloud distance:** Minimum Euclidean distance from pixel to nearest reprojected point cloud element.
- Each segment contributes equally (mean over segments).

All operations are tensor-based, enabling batched computation and GPU parallelisation.

### Iterative Optimisation
AdamW optimiser (lr=0.05, 2000 steps, one-cycle scheduling). Camera parameters initialised from coarse distributions covering main tribune cameras. Multiple initialisation with argmin selection outperforms single initialisation. Self-verification rejects samples where loss exceeds threshold $\tau$.

### Segment Localisation
DeepLabV3 ResNet-101 performs instance segmentation on all field segments. Post-processing selects representative points per segment (4 for lines, 8 for circles).

## Results

### SN-Calib-test-center (Calibration)
| Variant | AC@5 | AC@10 | AC@20 | CR | CS |
|---|---|---|---|---|---|
| TVCalib($\tau$), GT seg | 68.7 | 88.0 | 96.1 | 92.8 | **76.9** |
| TVCalib($\tau$), Pred seg | 57.6 | 81.7 | 93.2 | 93.7 | **72.6** |
| HDecomp + Chen&Little, GT | 53.7 | 77.5 | 88.4 | 80.3 | 65.1 |

### WC14-test (Calibration)
| Variant | AC@5 | AC@10 | AC@20 | CR | CS |
|---|---|---|---|---|---|
| TVCalib, GT seg | 64.4 | 86.7 | 96.0 | 100.0 | **86.4** |
| TVCalib, Pred seg | 39.9 | 71.9 | 90.5 | 100.0 | **75.0** |

TVCalib outperforms all baselines on both datasets. The gap between GT and predicted segmentation shows segment localisation is the main bottleneck — not the calibration module itself.

### IoU (Projection Error)
On WC14, TVCalib achieves IoU$_{part}$ of 96.1 (mean) / 97.1 (median), competitive with the best homography-based approaches without any fine-tuning on this dataset.

## Key Design Decisions

- **Segments over keypoints:** Using line/circle segments rather than keypoints provides denser supervision and avoids the keypoint visibility problem in broadcast views.
- **Single-step vs two-stage:** Direct camera parameter optimisation avoids error accumulation from homography decomposition.
- **No training data needed for calibration:** Only the segment localisation model requires training; the calibration module works purely via optimisation.
- **Lens distortion:** Joint optimisation of radial distortion coefficients ($k_1, k_2$) improves AC@5 on WC14 by +4.0% but can cause local minima at low FoV on SN-Calib.

## Relation to Other Vault Papers

TVCalib is used as a component in the [[soccernet-game-state-reconstruction|SoccerNet GSR baseline pipeline]] (Somers et al., 2024) for pitch localisation. The [[soccernet-game-state-reconstruction-improvement|Constructor Tech GSR pipeline]] (Golovkin et al., 2024) replaced TVCalib with a custom SegFormer-based regression approach for speed. The [[soccernet-v2-action-spotting|CCBV-SN approach]] (Cioppa et al., 2021) uses a different paradigm — synthetic template retrieval + STN refinement — rather than direct parameter optimisation.

## See Also

- [[camera-calibration]]
- [[game-state-reconstruction]]
- [[semantic-segmentation]]
