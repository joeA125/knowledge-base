---
title: "JaC Metric (ProCC)"
type: concept
tags: [computer-vision, evaluation, camera-calibration, sports-analytics]
sources: [raw/papers/camera-calibration-benchmarking.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-07
updated: 2026-07-23
---

# JaC Metric (ProCC)

JaC (Jaccard index for Camera Calibration) is a model-agnostic evaluation metric introduced by [[camera-calibration-benchmarking|Magera et al. (2025)]] as part of the ProCC benchmarking protocol for [[camera-calibration]] in sports. It is an application of the general [[jaccard-index]] to reprojected field elements.

## Motivation: Why IoU Is Inadequate

Existing metrics (IoU$_\text{whole}$, IoU$_\text{part}$) measure overlap between the projected field template and its deprojection-reprojection via annotated [[homography|homographies]]. This has three problems:

1. **Model-dependent:** Requires homography annotations, forcing all methods to be evaluated against a specific camera model regardless of what they actually compute.
2. **Blind to 3D:** Cannot evaluate reprojection of non-planar elements (goal posts, crossbars) since homographies only map the field plane.
3. **Unbounded reprojection error:** The traditional reprojection error ($\ell_2$ distance) ranges $[0, \infty)$, making it hard to interpret across methods.

## Definition

Given semantic polyline annotations of field elements in the image and estimated camera parameters with projection function $\pi$:

1. For each visible field element $L$, project its 3D model into the image: $\pi(L)$.
2. A field element is **correctly detected** (TP) if every annotated point $x_i$ is within $\tau$ pixels of the projected polyline: $\min_{S \in \pi(L)} d(x_i, S) < \tau, \;\forall x_i$.
3. Hallucinated or wrongly projected elements are FP. Missing elements are FN.

$$\text{JaC}_\tau = \frac{\text{TP}_\tau}{\text{TP}_\tau + \text{FN} + \text{FP}}$$

This is exactly the [[jaccard-index]] $|A \cap B| / |A \cup B|$ written in detection terminology, deliberately bridging camera calibration evaluation with object detection scoring.

## Key Properties

- **Model-agnostic:** Any camera model with a defined projection function $\pi(\mathbf{X}) \rightarrow \mathbf{x}$ can be evaluated — homography, pinhole, pinhole + [[radial-distortion]], fish-eye, etc.
- **Annotation-agnostic:** Ground truth consists of semantic point annotations along field markings (26 classes for soccer), not homographies. Annotations are valid for any camera model.
- **Interpretable thresholds:** $\tau = 5$ pixels differentiates between methods at current quality levels; $\tau = 2$ pixels separates precise methods. Results are proportions (0–100%), not unbounded distances.
- **Handles hallucination:** Unlike reprojection error, JaC penalises elements that are projected into the image when they shouldn't be visible.

The tolerance threshold $\tau$ is what adapts the Jaccard index — normally a hard set-membership criterion with no partial credit — to a continuous geometric problem. It defines what counts as a match *before* the set arithmetic begins.

## Relation to GS-HOTA

The [[game-state-reconstruction|GS-HOTA metric]] (from the SoccerNet GSR challenge) also uses a Jaccard-style formulation but operates at the athlete level (LocSim × IdSim). JaC operates at the field element level and evaluates only the camera calibration component, not tracking or identification.

## See Also

- [[jaccard-index]]
- [[camera-calibration-benchmarking|ProCC Source Summary]]
- [[camera-calibration]]
- [[homography]]
- [[radial-distortion]]
- [[game-state-reconstruction]]
