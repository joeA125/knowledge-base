---
title: "Sports Camera Calibration via Synthetic Data — Source Summary"
type: source_summary
tags: [computer-vision, deep-learning, sports-analytics, camera-calibration, generative-model]
sources: [raw/papers/sports-camera_calibration-synthetic_data.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-06-15
updated: 2026-06-15
---

# Sports Camera Calibration via Synthetic Data

**Authors:** Jianhui Chen, James J. Little
**Affiliation:** University of British Columbia
**Published:** 2019 (CVPR Workshop)

## Key Contribution

Proposes a sports [[camera-calibration]] method that uses a novel camera pose engine with only three significant free parameters ($f$, $\phi$, $\theta$) to generate large synthetic databases of edge images paired with known camera poses. A Siamese network learns compact 16-dimensional deep features for efficient retrieval, and a two-GAN model detects field markings from broadcast images. Achieves SOTA on the World Cup 2014 dataset without requiring annotated homographies for training.

## Method

### Camera Pose Engine
Decomposes the standard pinhole camera $P = KR[I | -C]$ into PTZ and base components: $P = K Q_\phi Q_\theta S_\rho S_{\phi'} [I | -C]$. By exploiting sports camera priors — cameras are roughly fixed on the main tribune, base tilt $\phi' \approx -90°$, roll $\rho \approx 0°$ — the model reduces to three significant free parameters: focal length $f$, pan $\theta$, and tilt $\phi$. These are uniformly sampled to generate 100K synthetic edge images.

### Deep Feature Extraction
A Siamese network (5 stride-2 convolutions + L2 normalisation) learns to embed edge images into a 16-dimensional feature space via contrastive loss. Pairs are labelled similar/dissimilar based on pan, tilt, and focal length thresholds.

### Two-GAN Field Marking Detection
Two chained conditional GANs (based on pix2pix):
1. **Segmentation GAN:** Segments the playing surface from the background (removes commercial boards, crowds).
2. **Detection GAN:** Detects field markings from the segmented foreground.
Soft alpha-blending boundaries prevent the detection GAN from memorising segmentation edges.

### Pose Retrieval and Refinement
Nearest-neighbour retrieval from the feature-pose database → Lucas-Kanade refinement on truncated distance images.

## Results

### World Cup 2014 Dataset
| Method | IoU$_{whole}$ (mean) | IoU$_{part}$ (mean) | IoU$_{part}$ (median) |
|---|---|---|---|
| DSM (Homayounfar 2017) | 83.0 | — | — |
| Dict. + HOG (Sharma 2018) | — | 91.4 | 92.7 |
| **Chen & Little (this)** | **89.4** | **94.5** | **96.1** |

### Robustness to Camera Displacement
On synthetic data, accuracy remains high (IoU$_{part}$ ≥ 92%) when camera displacement is within 5 metres. Performance degrades gracefully beyond that (~70% at 10m displacement).

### Component Analysis
- Segmentation GAN contributes +4.3% IoU$_{part}$
- LK warp refinement contributes +3.0%
- HOG features match deep features in accuracy (94.5%) but are 116× less compact (1860 vs 16 dimensions)

### Volleyball
Generalises to volleyball with minimal parameter changes: 97.6% mean IoU$_{part}$.

## Relation to Other Vault Papers

This method is used as a key baseline in [[tvcalib-camera-calibration-football|TVCalib]] (Theiner & Ewerth, 2023), which outperforms it by directly optimising camera parameters via a differentiable segment reprojection loss rather than retrieval + refinement. Also referenced as "CCBV" in [[soccernet-v2-action-spotting|Cioppa et al. (2021)]], who distilled a commercial tool into a similar retrieval-based architecture.

## See Also

- [[camera-calibration]]
- [[tvcalib-camera-calibration-football|TVCalib]]
- [[game-state-reconstruction]]
