---
title: "Sports Camera Calibration via Synthetic Data — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, camera-calibration, generative-model, metric-learning, gan]
sources: [raw/papers/sports-camera_calibration-synthetic_data.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  generated: 0%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-07-27
---

# Sports Camera Calibration via Synthetic Data

**Authors:** [[jianhui-chen]], [[james-little|James J. Little]]
**Affiliation:** University of British Columbia
**Published:** 2019 (CVPR Workshop)

## Key Contribution

A sports [[camera-calibration]] method using a camera pose engine with only three significant free parameters ($f$, $\phi$, $\theta$) to generate large synthetic databases of edge images paired with known poses. A [[siamese-network|Siamese network]] learns compact 16-dimensional features for retrieval, and a two-GAN model detects field markings from broadcast images. SOTA on World Cup 2014 **without requiring annotated homographies for training**.

## Method

### Camera Pose Engine

Decomposes the standard pinhole camera $P = KR[I | -C]$ into PTZ and base components: $P = K Q_\phi Q_\theta S_\rho S_{\phi'} [I | -C]$.

Exploiting sports camera priors — cameras roughly fixed on the main tribune, base tilt $\phi' \approx -90°$, roll $\rho \approx 0°$ — reduces this to three free parameters: focal length, pan, tilt. Uniform sampling generates 100K synthetic edge images.

**This is the paper's real idea.** The synthetic data is not a substitute for scarce annotation so much as a consequence of the domain having *so few genuine degrees of freedom* that the space can be enumerated. Compare [[theory-based-modelling]]: encoding a domain constraint as structure rather than learning it.

### Deep Feature Extraction

A [[siamese-network]] (5 stride-2 convolutions + L2 normalisation) embeds edge images into 16 dimensions via contrastive loss, with pairs labelled by pan, tilt and focal-length thresholds. See [[metric-learning]].

### Two-GAN Field Marking Detection

Two chained [[conditional-gan|conditional GANs]] (pix2pix-based):

1. **Segmentation GAN** — separates playing surface from background (commercial boards, crowds).
2. **Detection GAN** — detects field markings from the segmented foreground.

Soft alpha-blending boundaries prevent the detection GAN memorising segmentation edges.

### Pose Retrieval and Refinement

Nearest-neighbour retrieval from the feature-pose database, then Lucas-Kanade refinement on truncated distance images. See [[image-alignment]].

## Results

### World Cup 2014

| Method | IoU$_{whole}$ | IoU$_{part}$ mean | IoU$_{part}$ median |
|---|---|---|---|
| DSM (Homayounfar 2017) | 83.0 | — | — |
| Dict. + HOG (Sharma 2018) | — | 91.4 | 92.7 |
| **Chen & Little** | **89.4** | **94.5** | **96.1** |

### Robustness and Components

Accuracy holds (IoU$_{part}$ ≥ 92%) within 5 m of camera displacement, degrading to ~70% at 10 m. Segmentation GAN contributes +4.3%; LK refinement +3.0%.

**HOG features match deep features on accuracy (94.5%) but are 116× less compact** — 1,860 dimensions against 16. The learned representation buys *storage and retrieval speed*, not accuracy. A clean instance of the point on [[representation-learning]] that what a learned representation is *for* needs stating: here it is compression, and the paper is honest that accuracy was already available without it.

Generalises to volleyball with minimal parameter changes: 97.6% mean IoU$_{part}$.

## Relation to Other Vault Papers

A key baseline in [[tvcalib-camera-calibration-football|TVCalib]] (Theiner & Ewerth, 2023), which outperforms it by directly optimising camera parameters via a differentiable segment reprojection loss rather than retrieval plus refinement. Also referenced as "CCBV" in [[soccernet-v2-action-spotting|Cioppa et al. (2021)]].

⚠️ Its headline IoU numbers are computed under the **homography-based** evaluation that [[camera-calibration-benchmarking|ProCC]] argues is the wrong protocol — it rewards field registration while claiming to measure calibration. ProCC's consolidated leaderboard includes this result, so the comparison survives; the metric it was optimised against does not.

## See Also

- [[camera-calibration]] · [[homography]] · [[jac-metric]] · [[projective-geometry]] · [[radial-distortion]]
- [[siamese-network]] · [[metric-learning]] · [[conditional-gan]] · [[image-alignment]] · [[representation-learning]]
- [[jianhui-chen]] · [[james-little]] · [[game-state-reconstruction]]
- [[tvcalib-camera-calibration-football|TVCalib]] · [[camera-calibration-benchmarking|ProCC]] · [[soccernet-v2-action-spotting|CCBV-SN]]
