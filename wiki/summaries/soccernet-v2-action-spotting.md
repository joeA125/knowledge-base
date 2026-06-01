---
title: "Camera Calibration and Player Localization in SoccerNet-v2 for Action Spotting — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, camera-calibration, action-spotting]
sources: [raw/papers/soccernet-v2-action-spotting.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-05-26
updated: 2026-05-26
---

# Camera Calibration and Player Localization in SoccerNet-v2

**Authors:** Anthony Cioppa, Adrien Deliège, Floriane Magera, Silvio Giancola, Olivier Barnich, Bernard Ghanem, Marc Van Droogenbroeck
**Affiliations:** University of Liège, EVS Broadcast Equipment, KAUST
**Published:** 2021 (CVPR Workshop)

## Key Contribution

Distills a commercial camera calibration tool (EVS Xeebra) into a neural network (CCBV-SN) on 12,000 frames from SoccerNet's 500 games — the largest public soccer calibration dataset. Releases calibration estimates, player localisations, and the trained network. Proposes three calibration data representations and achieves SOTA on SoccerNet-v2 action spotting (Average-mAP **46.8%**).

## Calibration (CCBV-SN)

Based on the CCBV architecture (Sha et al., 2020): dictionary of synthetic field templates → zone segmentation (U-Net) → Siamese network for rough homography lookup → Spatial Transform Network for refinement. Student trained on 12K pseudo-ground-truth calibrations from the commercial teacher. Achieves 88.5% IoU (part) on World Cup 2014 without any fine-tuning on that dataset.

## Three Representations of Calibrated Data

1. **Top view images:** Color composition (CC) or binary channels (BC) showing field lines, camera polygon, and player squares.
2. **Feature vectors:** Top views compressed via frozen ResNet-34 or EfficientNet-B4.
3. **Player graph:** Nodes = players (RGB colour, world position, bbox area); edges between players within 25m. Processed by DeeperGCN (14 blocks).

## Action Spotting Integration

Split 17 SoccerNet-v2 classes into "patterned" (8 classes: penalty, kick-off, corner, throw-in, free-kick, cards) and "fuzzy" (9 classes: goal, substitution, offside, shots, etc.). Calibration features concatenated with ResNet features only for patterned classes within the CALF architecture. Best: CC + ResNet-34 + FCL → 46.8% Average-mAP, outperforming CALF alone (40.7%).

## See Also

- [[game-state-reconstruction]]
- [[soccernet-game-state-reconstruction|GSR Paper]]
