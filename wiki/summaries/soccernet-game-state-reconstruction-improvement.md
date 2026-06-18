---
title: "From Broadcast to Minimap: SOTA SoccerNet Game State Reconstruction — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, camera-calibration, object-detection]
sources: [raw/papers/soccernet-game-state-reconstruction-improvement.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-06-18
---

# From Broadcast to Minimap: SOTA SoccerNet Game State Reconstruction

**Authors:** Vladimir Golovkin, Nikolay Nemtsev, Vasyl Shandyba, et al.
**Affiliation:** Constructor Tech, Sofia
**Published:** 2024 (1st place, SoccerNet GSR Challenge 2024)

## Key Contribution

Presents the winning pipeline for the SoccerNet [[game-state-reconstruction]] Challenge 2024, achieving **GS-HOTA 63.81** — nearly 3× the original baseline (22.26) and 20+ points above the runner-up (43.15).

## Pipeline (Three Stages)

### 1. Raw Tracking (real-time, 80 FPS on RTX 3080Ti)
- **Detection:** Fine-tuned YOLOv5m.
- **[[camera-calibration|Camera parameters]]:** Custom SegFormer encoder-decoder predicting 7 params (x, y, z, pan, roll, tilt, FoV) with multi-loss training ($L^2_{world} + L^2_{camera} + L^1_{params} + L^2_{heatmap}$). Replaces [[tvcalib-camera-calibration-football|TVCalib]] from the baseline with direct regression.
- **Keypoints refinement:** ResNet-18 detecting 74 pitch keypoints; brute-force delta optimisation minimises reprojection error. Savitzky-Golay temporal smoothing.
- **Jersey number:** Modified ResNet-18 with dual classification heads (leading digit + second digit), trained with BinaryFocalLoss + CrossEntropy.
- **Feature extraction:** ResNet-50 ReID, OSNet TeamID (111 uniform classes), ResNet-18 orientation (4 classes), anomaly filter.
- **Tracking:** DeepSORT in world coordinates with orientation and TeamID constraints.

### 2. Team Detection
Cluster all TeamID embeddings (athletes within 30m of centre) into 3 groups (left team, right team, referees) via K-means. Goalkeepers detected separately by penalty-area filtering. Left/right assignment by per-frame voting.

### 3. Post-Processing
Split tracklets by jersey number/team conflicts → merge by jersey number (temporal non-overlap + physical feasibility) → merge by ReID similarity (mean vector + cross-multiplication pairwise comparison) → linear interpolation. Achieves ~90% tracklet reduction.

## Training Data
Combined real-world (22K images) and synthetic (40K images from Google Research Football Simulator with randomised textures/camera) for camera parameters. 62K total. Hyperparameters optimised with Optuna.

## See Also

- [[game-state-reconstruction]]
- [[camera-calibration]]
- [[homography]]
- [[soccernet-game-state-reconstruction|Original GSR Paper]]
- [[tvcalib-camera-calibration-football|TVCalib]]
