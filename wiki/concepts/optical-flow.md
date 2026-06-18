---
title: "Optical Flow"
type: concept
tags: [computer-vision, optical-flow, multi-object-tracking]
sources: [raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/detection-tracking-football-broadcast-footage.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-18
updated: 2026-06-18
---

# Optical Flow

Optical flow is the pattern of apparent motion of objects between consecutive video frames, caused by relative movement between the camera and the scene. It produces a vector field assigning a displacement $(dx, dy)$ to each pixel or feature point.

## Lucas-Kanade Method

The Lucas-Kanade (LK) algorithm (Lucas & Kanade, 1981) estimates optical flow by assuming brightness constancy and spatial coherence within a small local window around each tracked point. It solves for displacement via least-squares fitting of the image gradient equation:

$$I_x u + I_y v + I_t = 0$$

where $I_x, I_y, I_t$ are the image intensity gradients in $x$, $y$, and time.

**Pyramidal extension:** Because LK's local window assumption fails for large motions, the pyramidal implementation (Bouguet, 2001) applies LK at multiple resolutions — starting at coarse levels (large motions captured) and refining at fine levels (sub-pixel accuracy). This is the standard implementation in OpenCV.

## Use in Sports Analytics

### Object Tracking
[[amateur-football-analytics-computer-vision|Mavrogiannis (2021)]] uses pyramidal LK to track player bounding box midpoints between detection frames (every 5 frames). With a $50 \times 50$ window, 10 max iterations, and $\epsilon = 0.03$, a group of ~15 LK trackers processes a frame in ~0.02s — 3× faster than correlation-based tracking (~0.07s).

### Camera Pose Tracking
LK is also used for [[image-alignment|image alignment]] in camera calibration refinement. [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] cite LK for refining camera poses via distance image alignment (though they ultimately use ECC). The [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]] explored LK for camera motion tracking but found it unreliable because tracked corner points leave the frame during camera pans.

### Relation to Other Tracking Methods
Modern multi-object trackers like ByteTrack and DeepSORT (used in [[game-state-reconstruction]]) rely on detection-based association rather than optical flow. However, optical flow remains valuable for sparse, fast tracking between infrequent detections, and for camera motion compensation.

## See Also

- [[game-state-reconstruction]]
- [[camera-calibration]]
- [[amateur-football-analytics-computer-vision|Mavrogiannis Thesis]]
