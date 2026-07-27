---
title: "Semantic Segmentation"
type: concept
tags: [deep-learning, computer-vision, semantic-segmentation, probability-surface, weak-supervision, architecture]
sources: [raw/papers/context-aggregation-dilated-convolutions.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 42%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-07-27
---

# Semantic Segmentation

Semantic segmentation is the task of classifying each pixel in an image into one of a predefined set of categories. It is a **dense prediction** problem requiring both pixel-level accuracy and multi-scale contextual reasoning.

## Key Challenge

Classification networks achieve global context via pooling and subsampling but destroy spatial resolution. Semantic segmentation must maintain high-resolution output while reasoning about context at multiple scales — the two requirements pull in opposite directions, and essentially every architectural development in the area is an attempt to have both.

## Deep Learning Approaches

- **[[fully-convolutional-network|Fully Convolutional Networks]] (FCN):** Long et al. (2015) adapted classification networks for dense prediction using up-convolutions and skip connections, fusing predictions across resolutions in a "deep jet" hierarchy.
- **DeepLab:** Used [[dilated-convolution]]s to avoid resolution loss, combined with dense CRFs for structured prediction.
- **Dilated context module:** [[context-aggregation-dilated-convolutions|Yu & Koltun (2016)]] proposed a plug-in module using exponentially dilated convolutions for multi-scale context aggregation without resolution loss.

## Weakly Supervised Variants

Full segmentation labels are expensive — every pixel of every training image must be annotated. This has driven a line of work on learning dense output from sparser supervision:

| Supervision available | Approach |
|---|---|
| Every pixel labelled | Standard supervised training |
| One image-level label | Constrained CNNs (Pathak et al., 2015) |
| **One labelled location** | [[single-pixel-supervision]] |

The last is unusual and comes from outside vision. It is *easier* than image-level supervision in one respect — the label is tied to a known location, so there is no ambiguity about which prediction it constrains — and harder in another, since coverage of the output space is far sparser.

## Transfer Outside Vision

The machinery generalises to any domain with a **regular grid and translation-equivariant structure**, where the same local pattern means the same thing wherever it appears.

The clearest example in this vault is [[soccermap]], which discretises a football pitch to $104 \times 68$ cells, renders tracking data as stacked spatially-registered layers, and applies FCN machinery unchanged to produce full-pitch [[probability-surface|value and probability surfaces]]. The multi-scale fusion structure is taken directly from Long et al.

Two differences from the vision setting are worth noting, because they show where the analogy strains:

- **Supervision is far sparser.** One labelled cell per example rather than all of them.
- **The domain is not fully translation-invariant.** Images generally are; a pitch is not, since the goals break the symmetry. The framework recovers this by supplying distance-to-goal and angle-to-goal as explicit input layers rather than expecting the architecture to learn it.

## Evaluation

The standard metric is mean Intersection over Union (mIoU) across classes — see [[jaccard-index]], which also underlies the JaC metric used for [[camera-calibration]] evaluation. Common benchmarks include Pascal VOC 2012, Cityscapes, CamVid, and KITTI.

Note that mIoU has no counterpart in the football surface setting, since there is no ground-truth surface to intersect with. Surface models are instead assessed by [[probability-calibration|calibration]] and loss at observed locations only.

## See Also

- [[fully-convolutional-network]] · [[dilated-convolution]] · [[feature-pyramid-network]]
- [[probability-surface]] · [[single-pixel-supervision]] · [[soccermap]]
- [[jaccard-index]] · [[convolution]] · [[residual-connections]] · [[batch-normalization]]
- [[context-aggregation-dilated-convolutions|Dilated Convolutions Summary]]
