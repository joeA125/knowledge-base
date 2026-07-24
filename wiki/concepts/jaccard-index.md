---
title: "Jaccard Index"
type: concept
tags: [statistics, evaluation, metric-learning, machine-learning, clustering]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/camera-calibration-benchmarking.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-24
---

# Jaccard Index

The Jaccard index measures the similarity of two sets as the size of their intersection over the size of their union:

$$J(A, B) = \frac{|A \cap B|}{|A \cup B|}$$

It ranges from 0 (disjoint) to 1 (identical), and is sometimes called *intersection over union* (IoU). The Jaccard distance $1 - J(A,B)$ is a proper metric.

## Why It Is Useful

The union in the denominator penalises both kinds of error symmetrically: elements in $A$ but not $B$, and elements in $B$ but not $A$. This makes it a natural choice when both false positives and false negatives matter and there is no meaningful "negative class" to count — a situation common in detection and retrieval, where the set of things that *could* have been retrieved is unbounded.

## Uses in This Vault

### Comparing player rankings
[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] use it to compare top-$k$ player rankings from [[expected-threat|xT]] and [[vaep]]:

| $k$ | 5 | 25 | 50 | 100 | 200 | 300 |
|---|---|---|---|---|---|---|
| $J$ | 0.18 | 0.48 | 0.35 | 0.55 | 0.75 | 0.85 |

The non-monotonicity is informative: similarity rises to $k = 25$, dips at $k = 50$, then climbs steadily. Beyond the top ~25 players most receive similar average ratings, so tiny rating differences reshuffle rankings substantially. Jaccard similarity over set membership is insensitive to *ordering*, which is precisely what makes it robust to that churn.

### Camera calibration evaluation
The [[jac-metric|JaC metric]] from the [[camera-calibration-benchmarking|ProCC paper]] is a Jaccard index over reprojected field elements:

$$\text{JaC}_\tau = \frac{\text{TP}_\tau}{\text{TP}_\tau + \text{FN} + \text{FP}}$$

which is exactly $|A \cap B| / |A \cup B|$ written in detection terminology.

### Segmentation and detection
The IoU form is the standard overlap criterion in [[object-detection]] and [[semantic-segmentation]], including the IoU$_{whole}$ and IoU$_{part}$ measures used across the sports [[camera-calibration]] literature.

## Comparing Partitions, Not Sets

Jaccard compares two *sets*. Comparing two **partitions** of the same objects — as in [[clustering]] evaluation — needs a different construction, because cluster labels are arbitrary and a partition is a family of sets rather than one.

The [[adjusted-rand-index]] fills that role. It operates on object *pairs* rather than elements, is invariant to relabelling, and is corrected for chance agreement — a correction Jaccard lacks. See that page for the comparison.

## Limitations

- **Ignores ordering.** Two top-25 lists with identical membership but reversed order score 1.0. Rank correlation (Spearman, Kendall) captures what Jaccard discards.
- **No chance correction.** Unlike the [[adjusted-rand-index]], a Jaccard score is not calibrated against what random agreement would produce.
- **Sensitive to set size.** Small sets give coarse, high-variance values — hence $J = 0.18$ at $k=5$, where a single differing player moves the score substantially.
- **No partial credit.** Membership is binary; near-misses count as full misses. The [[jac-metric|JaC metric]] works around this with a tolerance threshold $\tau$ that defines what counts as a match before the set arithmetic begins.

## See Also

- [[adjusted-rand-index]]
- [[jac-metric]]
- [[expected-threat]]
- [[vaep]]
- [[object-detection]]
- [[camera-calibration]]
