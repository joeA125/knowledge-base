---
title: "Object Detection"
type: concept
tags: [object-detection, computer-vision, deep-learning]
sources: [raw/papers/computer-vision-football-review.md, raw/papers/amateur_footbal_analytics_computer_vision.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-07
updated: 2026-07-07
---

# Object Detection

Object detection is the computer-vision task of locating and classifying objects within an image, typically by predicting a bounding box and a class label for each instance. It combines localisation (where) with classification (what), and is the foundation of downstream tasks like tracking and [[game-state-reconstruction]].

## Families of Detectors

- **Two-stage:** a region-proposal stage followed by classification/refinement (e.g. Faster R-CNN). Generally higher accuracy, slower.
- **One-stage:** a single network predicts boxes and classes directly (e.g. the YOLO family, SSD). Generally faster, favoured for real-time use.^[inferred: standard taxonomy; the ingested sources focus on specific detectors]

Multi-scale feature fusion via a [[feature-pyramid-network]] is a common ingredient, letting a detector find both large and small objects from one backbone.

## Use in Football Analytics

Detecting players and the ball is the entry point of most sports-video pipelines. The [[computer-vision-football-review|CV football review]] (Zheng et al., 2025) reports the YOLO family dominating football detection work. The [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]] instead uses FootAndBall, a lightweight [[feature-pyramid-network|FPN]]-based detector, exploiting multi-scale feature maps to handle the extreme size gap between players and the ball. Small, fast-moving ball detection remains one of the hardest cases.

## See Also

- [[feature-pyramid-network]]
- [[game-state-reconstruction]]
- [[camera-calibration]]
- [[computer-vision-football-review|Source Summary]]
