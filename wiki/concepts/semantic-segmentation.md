---
title: "Semantic Segmentation"
type: concept
tags: [deep-learning, computer-vision, semantic-segmentation]
sources: [raw/papers/context-aggregation-dilated-convolutions.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: draft
created: 2026-05-08
updated: 2026-05-08
---

# Semantic Segmentation

Semantic segmentation is the task of classifying each pixel in an image into one of a predefined set of categories. It is a dense prediction problem requiring both pixel-level accuracy and multi-scale contextual reasoning.

## Key Challenge

Classification networks achieve global context via pooling/subsampling but destroy spatial resolution. Semantic segmentation must maintain high-resolution output while reasoning about context at multiple scales.

## Deep Learning Approaches

- **Fully Convolutional Networks (FCN):** Long et al. (2015) adapted classification networks for dense prediction using up-convolutions and skip connections.
- **DeepLab:** Used [[dilated-convolution]]s to avoid resolution loss, combined with dense CRFs for structured prediction.
- **Dilated context module:** [[context-aggregation-dilated-convolutions|Yu & Koltun (2016)]] proposed a plug-in module using exponentially dilated convolutions for multi-scale context aggregation without resolution loss.

## Evaluation

The standard metric is mean Intersection over Union (mIoU) across classes. Common benchmarks include Pascal VOC 2012, Cityscapes, CamVid, and KITTI.

## See Also

- [[dilated-convolution]]
- [[residual-connections]]
- [[batch-normalization]]
