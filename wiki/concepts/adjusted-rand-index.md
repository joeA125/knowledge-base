---
title: "Adjusted Rand Index (ARI)"
type: concept
tags: [clustering, evaluation, statistics, machine-learning, mixture-model]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 42%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Adjusted Rand Index (ARI)

The ARI (Hubert & Arabie, 1985) measures agreement between two partitions of the same objects, corrected for the agreement expected by chance:

$$\text{ARI} = \frac{\text{Index} - \mathbb{E}[\text{Index}]}{\max(\text{Index}) - \mathbb{E}[\text{Index}]}$$

It equals 1 for identical partitions, approximately 0 for a random partition, and can be negative when agreement is *worse* than chance.

## Why Chance Correction Matters

The unadjusted Rand Index counts the proportion of object *pairs* on which two partitions agree — either co-clustered in both or separated in both. Its problem is that this proportion is high by default: with many clusters, most pairs are separated in both partitions, so the raw index approaches 1 regardless of quality.

The adjustment subtracts the expected index under a null model where partitions are random with the observed cluster sizes fixed. This is the same move as Cohen's kappa for classification agreement, and the same motivation as using [[jaccard-index]] over raw accuracy when one class dominates.

## Relation to Jaccard Index

Both are set-comparison measures, operating at different levels:

| | [[jaccard-index]] | ARI |
|---|---|---|
| Compares | Two sets | Two *partitions* of the same objects |
| Unit | Elements | Object pairs |
| Chance-corrected | No | **Yes** |
| Range | $[0, 1]$ | $(-1, 1]$ |
| Label-dependent | Yes | No — invariant to relabelling |

ARI's label-invariance is essential in clustering, where cluster indices are arbitrary — the same property that makes [[identifiability|label switching]] harmless for mixture-based clustering.

## Use in the Football Mixture Study

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] use ARI to validate their [[mixture-model]] on simulated data, where ground-truth partitions are known:

| Separation | $n=50$ | $n=100$ | $n=200$ | $n=400$ |
|---|---|---|---|---|
| Easy | 0.642 | 0.770 | 0.829 | 0.833 |
| Intermediate | 0.444 | 0.506 | 0.605 | 0.698 |
| Hard | 0.356 | 0.378 | 0.431 | 0.524 |

Recovery improves with sample size, consistent with maximum-likelihood consistency. But the easy case plateaus near 0.83 rather than climbing toward 1 — the simulation's stated 5% Bayes classification error puts a ceiling on achievable ARI, so this is close to the best attainable rather than a shortfall of the estimator.

That distinction is easy to miss and worth stating generally: **ARI should be read against the achievable maximum given class overlap, not against 1.**

## The Ground-Truth Problem

ARI requires a reference partition, which is why it appears in the simulation study and not in the real-data analysis. On real football possessions there is no true clustering to compare against — the same absence of ground truth that drives [[split-half-reliability]] and [[predictive-validity]] in the action-valuation literature.

Real-data clustering is therefore validated differently: internal criteria ([[mixture-model|BIC]]), stability under resampling, or external interpretability — here, whether the recovered clusters correspond to recognised tactical patterns like counter-attacking and positional play.

## See Also

- [[mixture-model]]
- [[jaccard-index]]
- [[identifiability]]
- [[split-half-reliability]]
- [[football-event-sequences-point-process-mixture|Source Summary]]
