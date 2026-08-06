---
title: "ProCC: A Universal Protocol to Benchmark Camera Calibration for Sports — Source Summary"
type: source_summary
tags: [computer-vision, sports-analytics, camera-calibration, evaluation, projective-geometry, radial-distortion]
sources: [raw/papers/camera-calibration-benchmarking.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  generated: 0%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-07
updated: 2026-07-27
---

# ProCC: A Universal Protocol to Benchmark Camera Calibration for Sports

**Authors:** [[floriane-magera]], Thomas Hoyoux, Olivier Barnich, [[marc-van-droogenbroeck]]
**Affiliations:** EVS Broadcast Equipment, University of Liège
**Published:** 2025 (CVPR)

## Key Contribution

Introduces ProCC, a model-agnostic benchmarking protocol for [[camera-calibration]] in sports, based on two principles: (1) the protocol should be agnostic to the camera model, and (2) evaluation should use reprojection of accurately known 3D objects rather than [[homography]]-based IoU metrics. Introduces the [[jac-metric|JaC]] (Jaccard index for Camera Calibration) metric and demonstrates that richer camera models incorporating [[radial-distortion]] significantly outperform homography-based approaches.

## Core Argument: Field Registration ≠ Camera Calibration

The paper draws a sharp distinction between two tasks that are often conflated:

- **Sports field registration:** Estimates a [[homography]] mapping the 2D field plane to the image. Limited to planar correspondences — cannot model 3D elements (goal posts, corner flags, airborne balls) or [[radial-distortion|lens distortion]].
- **Camera calibration:** Estimates full camera parameters (intrinsics + extrinsics) providing a mapping between the 3D world and the image. Supports all downstream applications including offside detection, 3D tracking, and augmented reality.

Most existing benchmarks (World Cup 2014, CARWC) annotate homographies as ground truth, which forces and rewards field registration methods even when papers claim to do camera calibration.

## ProCC Protocol

### Annotations
Instead of homographies, ProCC uses semantic point annotations along field markings — polylines labelled by field element class (26 classes for soccer). These annotations are camera-model-agnostic: they describe what is observed in the image, not a specific mathematical mapping.

### JaC Metric
For each visible field element $L$, project its 3D model into the image using estimated camera parameters $\pi$. A field element is "correctly detected" if every annotated point along its polyline is within $\tau$ pixels of the projected polyline. Then:

$$\text{JaC}_\tau = \frac{\text{TP}_\tau}{\text{TP}_\tau + \text{FN} + \text{FP}}$$

where FP includes hallucinated elements and wrongly projected elements. This bridges camera calibration evaluation with object detection scoring. See [[jac-metric]].

## Key Results

### World Cup 2014 — Camera Model Comparison
| Camera Model | JaC₅ | Reprojection Error |
|---|---|---|
| Homography (WC14 annotations) | 67.4 | 3.07 |
| Homography (CARWC annotations) | 79.1 | 1.79 |
| **Pinhole + radial distortion (Xeebra)** | **92.5** | **1.44** |

The 13.4-point gap between CARWC homographies and pinhole + distortion demonstrates that the limitation is the camera model, not the annotation quality.

### SoccerNet — Radial Distortion Impact
| Model | JaC₅ | JaC₂ | Reproj. Error |
|---|---|---|---|
| Pinhole only (P) | 78.7 | 40.2 | 4.51 |
| Pinhole + radial (R) | **83.1** | **54.3** | **4.01** |

On the larger, more diverse SoccerNet dataset, radial distortion is a necessity — improving JaC₂ by 14.1 points.

### Practical Impact
Disagreement between homography and pinhole+distortion models exceeds **2.5 metres** in some pitch regions. This exceeds FIFA's 50cm accuracy requirement for Semi-Automated Offside Technology, meaning homography-based methods are fundamentally inadequate for professional refereeing applications.

## Relation to Other Vault Papers

This paper directly critiques the evaluation methodology used by [[tvcalib-camera-calibration-football|TVCalib]] (Theiner & Ewerth, 2023) — cited as having "showcased some discrepancies" between ProCC and WC14 evaluation. The [[soccernet-v2-action-spotting|CCBV-SN]] approach (Cioppa et al., 2021) is by the same University of Liège group; [[floriane-magera|Magera]], Barnich and [[marc-van-droogenbroeck|Van Droogenbroeck]] are co-authors on both. The [[sports-camera-calibration-synthetic-data|Chen & Little (2019)]] results on WC14 (IoU$_{part}$ 94.5) appear in the consolidated leaderboard.

**A benchmarking paper, in a vault where almost nothing benchmarks.** Worth noting against `no-cross-framework-benchmarking`, declared on [[action-valuation-frameworks-compared]]: that claim concerns the *football valuation* literature, and the computer-vision side of this vault behaves entirely differently. ProCC exists precisely to make cross-method comparison possible, and consolidates a leaderboard across groups. The valuation literature has no equivalent.

That contrast is worth holding onto — it shows the benchmarking gap is a property of one research community rather than of sports analytics as a field.

## See Also

- [[camera-calibration]] · [[jac-metric]] · [[radial-distortion]] · [[homography]] · [[projective-geometry]]
- [[game-state-reconstruction]] · [[multi-object-tracking]] · [[object-detection]]
- [[floriane-magera]] · [[marc-van-droogenbroeck]]
- [[tvcalib-camera-calibration-football|TVCalib]] · [[sports-camera-calibration-synthetic-data|Chen & Little]] · [[soccernet-v2-action-spotting|CCBV-SN]]
