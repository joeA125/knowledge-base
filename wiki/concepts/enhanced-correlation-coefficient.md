---
title: "Enhanced Correlation Coefficient Maximization"
type: concept
tags: [computer-vision, image-alignment, camera-calibration, sports-analytics]
sources: [raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/sports-camera_calibration-synthetic_data.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-18
updated: 2026-06-18
---

# Enhanced Correlation Coefficient Maximization

Enhanced Correlation Coefficient Maximization (ECC; Evangelidis & Psarakis, 2008) is an $L_2$-based iterative image alignment algorithm that estimates the geometric transformation (affine, [[homography]], etc.) mapping a source image to a target image.

## Key Advantages

1. **Photometric invariance:** Unlike pixel intensity difference methods, ECC is invariant to linear changes in contrast and brightness between images. This makes it robust to lighting variation — critical for sports broadcast where lighting shifts during a game.
2. **Efficient linearisation:** Although the objective function is nonlinear in the transformation parameters, the iterative scheme reduces each step to a linear problem, making it computationally efficient despite its accuracy.

## Use in Camera Pose Tracking

ECC is the workhorse for temporal camera pose propagation in the sports [[camera-calibration]] pipeline:

### Estimation Phase (Initial Refinement)
After [[siamese-network|Siamese retrieval]] returns an approximate camera pose from the database, ECC aligns the retrieved edge map to the Court Detector's observed edge map. [[amateur-football-analytics-computer-vision|Mavrogiannis (2021)]] uses 1000 iterations with $\epsilon = 10^{-4}$ for this expensive but accurate refinement.

**Distance transformation** is applied to both edge maps before ECC, converting them to float images where pixel values represent distance from the nearest edge. This smooths the objective landscape, improving convergence speed without sacrificing accuracy. A distance threshold of 15 caps maximum pixel values.

### Tracking Phase (Frame-to-Frame)
For subsequent frames, ECC aligns the previous frame's estimated edge map to the current frame's detected edge map — avoiding the expensive database search entirely. Only 50 iterations with $\epsilon = 10^{-3}$ are needed because consecutive frames differ minimally. Output is stabilised via weighted-average homography history.

### Comparison with Lucas-Kanade
The [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]] tested both ECC and pyramidal [[optical-flow|Lucas-Kanade]] for camera tracking. LK failed because tracked corner points exit the frame during pans; ECC operates on the full edge map and handles camera motion more robustly.

## See Also

- [[camera-calibration]]
- [[homography]]
- [[optical-flow]]
- [[siamese-network]]
