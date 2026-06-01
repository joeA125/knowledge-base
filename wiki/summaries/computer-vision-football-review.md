---
title: "A Review of Computer Vision Technology for Football Videos — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, multi-object-tracking, object-detection, action-spotting]
sources: [raw/papers/computer-vision-football-review.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-06-01
updated: 2026-06-01
---

# A Review of Computer Vision Technology for Football Videos

**Authors:** Fucheng Zheng, Duaa Zuhair Al-Hamid, Peter Han Joo Chong, Cheng Yang, Xue Jun Li
**Affiliations:** Auckland University of Technology, Zyetric Technologies
**Published:** 2025 (Information, 16(5), 355)

## Key Contribution

A comprehensive PRISMA-guided review of CV applications in football video analysis (2020–2024), covering 49 studies across four "ultimate purposes": player/ball detection and tracking, motion prediction, tactical analysis, and event detection. Football accounts for 49% of CV sports research yet lacks dedicated comprehensive reviews — this paper fills that gap.

## Four Ultimate Purposes

### 1. Player/Ball Detection and Tracking
Dominated by YOLO variants (v3–v8) combined with tracking algorithms (SORT, DeepSORT, ByteTrack). Key challenge: ball detection remains poor due to small size, rapid motion, and occlusion. YOLO's grid-based architecture and downsampling inherently struggle with small objects. Best results: YOLOv4+SORT (F1 0.94 for players); ball recall remains low across all methods.

### 2. Motion Prediction
Predicting ball trajectories, pass receivers, and penalty outcomes. Notable approaches: Transformer encoder with trajectory+visual data for pass prediction (top-1 accuracy 62.5%); YOLOv4+LSTM for penalty prediction (79% accuracy at 1s before kick). Key limitation: models lack contextual game information (score, time remaining).

### 3. Tactical Analysis
Formation analysis, ball possession, pass success, and offside detection. Approaches range from big data+LSTM for decision-making to YOLOv2+DeepSORT+Triplet CNN-DCGAN for automated tactical statistics (97.46% team identification accuracy). Key challenge: adapting to dynamic in-game strategy changes.

### 4. Event Detection
Spotting goals, cards, fouls, celebrations from video. Multi-modal approaches (visual+audio) outperform visual-only: I3D-NL achieves 92.48% top-1 accuracy for event recognition; VGGish achieves 94.4% for commentator excitement classification. Trade-off: multimodal is more accurate but more computationally expensive.

## Research Trends (2020–2024)

A notable shift from detection/tracking research (declining from 2023) toward motion prediction, tactical analysis, and event detection (all increasing). This suggests the field is moving from foundational object-level tasks toward higher-level decision-making and understanding.

## Key Challenges Identified

1. Low-resolution imagery of distant players and balls
2. Motion blur during rapid movements
3. Complex occlusions in crowded scenes
4. Lack of large-scale annotated football-specific datasets
5. Computational cost of real-time processing
6. Model bias toward common play types (e.g., short passes)
7. Sensitivity to environmental variations (lighting, weather, camera angles)

## Future Directions

- Super-resolution techniques for improved small-object detection
- Collaborative dataset-building efforts across organisations
- Integration of contextual game information (score differentials, time remaining)
- More robust algorithms handling environmental variation

## See Also

- [[game-state-reconstruction]]
- [[soccernet-game-state-reconstruction|SoccerNet GSR]]
- [[detection-tracking-football-broadcast-footage|Single-camera detection/tracking]]
