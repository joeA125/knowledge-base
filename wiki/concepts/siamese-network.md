---
title: "Siamese Network"
type: concept
tags: [deep-learning, architecture, metric-learning, representation-learning]
sources: [raw/papers/sports-camera_calibration-synthetic_data.md, raw/papers/amateur_footbal_analytics_computer_vision.md, raw/papers/soccernet-v2-action-spotting.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Siamese Network

A Siamese network (Bromley et al., 1993; Hadsell et al., 2006) consists of two identical sub-networks ("branches") sharing the same weights. Each branch maps an input to an embedding, and a distance metric (typically Euclidean) in the embedding space measures similarity between the two inputs. The network is trained with contrastive loss:

$$\mathcal{L}(x_1, x_2, y) = y \cdot D_w(x_1, x_2) + (1 - y) \cdot \max(0, m - D_w(x_1, x_2))$$

where $D_w = \|f_w(x_1) - f_w(x_2)\|_2^2$, $y \in \{0, 1\}$ indicates similar/dissimilar, and $m$ is a margin.

## Use in Sports Camera Calibration

Siamese networks are central to the retrieval-based [[camera-calibration]] paradigm in this vault:

- [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] train a Siamese network on pairs of synthetic edge maps to learn 16-dimensional embeddings. Similar camera poses (pan/tilt/focal length within thresholds) produce similar embeddings. At inference, the edge map from a broadcast frame is embedded and the nearest neighbour in a 90K feature-pose database is retrieved via FLANN.
- [[amateur-football-analytics-computer-vision|Mavrogiannis (2021)]] reuses the same Siamese architecture (5 stride-2 convolutions → 16-d output) and training procedure for their camera estimator module.
- [[soccernet-v2-action-spotting|CCBV-SN (Cioppa et al., 2021)]] uses a Siamese network in a similar retrieval paradigm to match segmented field images to synthetic templates.

## Key Properties

- **Weight sharing** ensures both inputs are mapped by the same function, making similarity symmetric.
- **Compact embeddings:** Chen & Little's 16-d features are 116× more compact than HOG features (1860-d) while matching accuracy.
- **Data efficiency:** Training requires only labelled pairs (similar/dissimilar), not class labels, and pairs can be generated synthetically from known camera distributions.

## Variants

- **Triplet networks** use anchor-positive-negative triples instead of pairs (used in ReID tasks within [[game-state-reconstruction]]).
- **Contrastive learning** generalises the Siamese framework to self-supervised pre-training (SimCLR, MoCo).

## See Also

- [[camera-calibration]]
- [[sports-camera-calibration-synthetic-data|Chen & Little Source Summary]]
- [[attention-mechanism]]
