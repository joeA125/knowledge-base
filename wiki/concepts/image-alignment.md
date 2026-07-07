---
title: "Image Alignment"
type: concept
tags: [image-alignment, computer-vision, projective-geometry]
sources: [raw/papers/sports-camera-calibration-synthetic-data.md, raw/papers/amateur_footbal_analytics_computer_vision.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-07
updated: 2026-07-07
---

# Image Alignment

Image alignment (image registration) is the task of finding the geometric transformation that best maps one image onto another — for instance warping a current video frame onto a reference view, or a model template onto an observed image. It underpins camera-motion estimation, panorama stitching, and camera-pose refinement in sports analytics.

## Approaches

- **Feature-based:** detect and match keypoints, then estimate a transformation (often a [[homography]]) from the correspondences via robust fitting.
- **Intensity/direct:** iteratively adjust the transformation to maximise photometric agreement between the images, without explicit feature matching. [[optical-flow|Lucas-Kanade]] warping and the [[enhanced-correlation-coefficient|Enhanced Correlation Coefficient (ECC)]] method are direct approaches.

## Use in Camera Calibration

Image alignment is central to refining camera pose in broadcast sports video. [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] refine an initial camera estimate by aligning a rendered field model with the detected field, ultimately using [[enhanced-correlation-coefficient|ECC]] because it is invariant to photometric distortions. [[optical-flow|Lucas-Kanade]] alignment was also explored for tracking camera motion in the [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]], but proved unreliable when tracked points leave the frame during camera pans.^[inferred: synthesis drawn from the related optical-flow and camera-calibration pages]

## See Also

- [[enhanced-correlation-coefficient]]
- [[optical-flow]]
- [[camera-calibration]]
- [[homography]]
- [[sports-camera-calibration-synthetic-data|Source Summary]]
