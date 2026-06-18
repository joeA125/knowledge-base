---
title: "Feature Pyramid Network"
type: concept
tags: [deep-learning, architecture, object-detection, computer-vision]
sources: [raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/computer-vision-football-review.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-18
updated: 2026-06-18
---

# Feature Pyramid Network

The Feature Pyramid Network (FPN; Lin et al., 2017) is an architecture pattern for building multi-scale feature representations in convolutional networks. It exploits the inherent pyramidal hierarchy of deep CNNs — where earlier layers have high spatial resolution but low-level features, and later layers have low spatial resolution but high-level semantic features — by adding a top-down pathway with lateral connections.

## Architecture

1. **Bottom-up pathway:** Standard forward pass through a backbone (e.g., ResNet), producing feature maps at decreasing spatial resolutions ($C_2, C_3, C_4, C_5$).
2. **Top-down pathway:** Upsamples coarser feature maps (2× nearest-neighbour) and adds them element-wise to the corresponding bottom-up feature map via $1 \times 1$ convolution lateral connections.
3. **Output:** Multi-scale feature maps ($P_2, P_3, P_4, P_5$) that combine high-resolution spatial detail with high-level semantics at every scale.

## Why It Matters for Object Detection

Traditional detectors use only the final (coarsest) feature map, losing fine spatial detail needed for small objects. FPN produces semantically rich features at all scales with marginal extra cost, enabling detection of both large and small objects from the same backbone.

## Use in Sports Analytics

### FootAndBall Detector
The [[amateur-football-analytics-computer-vision|Mavrogiannis thesis]] uses FootAndBall (Komorowski et al., 2019), a lightweight FPN-based detector with only 198K parameters. Five convolutional blocks produce feature maps at decreasing resolutions; the top-down pathway merges them via $1 \times 1$ convolutions. Ball detection uses high-resolution feature maps ($w/4 \times h/4$) while player detection uses coarser maps ($w/16 \times h/16$), exploiting FPN's multi-scale design to handle the extreme size difference between players and the ball.

### YOLO Variants
The YOLO family (v3+) that dominates the [[computer-vision-football-review|CV football review]] implicitly uses FPN-style multi-scale feature fusion. YOLOv3 introduced multi-scale detection heads at three different feature map resolutions, following the FPN design pattern.

## Relation to Other Architectures

- FPN is complementary to [[residual-connections]] — ResNet provides the backbone; FPN adds the multi-scale fusion.
- U-Net (used in Pix2Pix / [[conditional-gan]]) uses a similar encoder-decoder structure with skip connections, but for dense pixel-wise prediction rather than detection.

## See Also

- [[object-detection]]
- [[residual-connections]]
- [[conditional-gan]]
- [[game-state-reconstruction]]
