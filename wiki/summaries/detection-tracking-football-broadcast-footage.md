---
title: "Multi-Class Detection and Tracking in Soccer Broadcast Footage — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, object-detection]
sources: [raw/papers/detection-tracking-football-broadcast-footage.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-05-26
---

# A Computer Vision Framework for Multi-Class Detection and Tracking in Soccer Broadcast Footage

**Author:** Daniel Tshiani
**Published:** 2025

## Key Contribution

Develops and evaluates an accessible single-camera pipeline for detecting and tracking players, goalkeepers, referees, and the ball from standard broadcast footage, demonstrating that meaningful analytics can be extracted without expensive multi-camera or GPS systems. Aimed at making soccer analytics affordable for colleges, academies, and amateur clubs.

## Pipeline

YOLOv8s (fine-tuned, 1280×1280 input) → CLIP embeddings for appearance → UMAP dimensionality reduction → K-Means team clustering → ByteTrack for temporal association.

## Results (mAP@0.5)

| Class | Precision | Recall | mAP@0.5 | mAP@0.5–0.95 |
|---|---|---|---|---|
| Player | 0.957 | 0.978 | 0.993 | 0.767 |
| Referee | 0.864 | 0.925 | 0.946 | 0.610 |
| Goalkeeper | 0.801 | 0.897 | 0.931 | 0.659 |
| Ball | 0.914 | 0.511 | 0.616 | 0.296 |
| **All** | **0.884** | **0.828** | **0.871** | **0.583** |

Ball detection is the primary weakness (high precision but low recall due to small size, rapid motion, and frequent occlusion).

## Limitations

- ByteTrack IDs are randomly assigned (no jersey number correspondence) and reset on re-entry.
- Team classification degrades under lighting changes; per-frame clustering causes label flipping.
- Trained only on broadcast footage; generalises poorly to tactical/drone/sideline views.

## See Also

- [[game-state-reconstruction]]
