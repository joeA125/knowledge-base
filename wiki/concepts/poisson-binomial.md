---
title: "Poisson Binomial Distribution"
type: concept
tags: [statistics, probabilistic-classification, expected-versus-actual, evaluation, uncertainty-quantification, sports-analytics]
sources: [raw/papers/physics_based_pass_probabilities.md]
confidence: 0.85
provenance:
  extracted: 45%
  inferred: 30%
  generated: 5%
  imported: 20%
  ambiguous: 0%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Poisson Binomial Distribution

The distribution of the number of successes in $n$ independent Bernoulli trials with **different** success probabilities $p_1, \dots, p_n$.

The binomial distribution is the special case where all $p_i$ are equal.^[imported: standard statistical background, not from any held source] Its mean and variance are pleasantly simple:

$$\mu = \sum_{i=1}^{n} p_i \qquad \sigma^2 = \sum_{i=1}^{n} p_i(1 - p_i)$$

The full probability mass function is awkward — a sum over subsets, exponential in $n$ — but for the use below only the first two moments are needed.

## Why It Matters Here

It is **the aggregation mechanism for any per-event probability model.**

A model like [[pass-probability-model|the pass probability model]] outputs a different $p_i$ for every pass. To ask a season-level question — did this player receive more passes than expected? — those probabilities must be aggregated, and they cannot simply be averaged and multiplied by $n$ without discarding the variance structure.

The Poisson binomial supplies both moments directly, which turns a collection of per-event predictions into a **testable expectation with an uncertainty attached.**

## The Pattern It Enables

$$\text{observed count} - \sum_i p_i, \quad \text{scaled by} \ \sqrt{\textstyle\sum_i p_i(1-p_i)}$$

More successes than expected suggests something the model does not capture — skill, effort, positioning. Fewer suggests the opposite. Dividing by the standard deviation says whether the gap is larger than chance.

[[receiving-efficiency]] is the vault's worked instance. The general form is **expected-versus-actual counting**, which recurs far beyond sport:^[imported: cross-domain parallels are background knowledge, not from any held source] risk-adjusted mortality in medicine (observed deaths against those predicted from case mix), value-added models in education, and expected-loss models in insurance all compute the same quantity from a per-case probability model.

## The Independence Assumption

Both moments above require the trials to be **independent**. That is where the framework strains in practice, and the strain is usually unexamined.

Passes within a match are not independent: a team under sustained pressure produces many low-probability passes in sequence, and a player's receptions are correlated with his team's possession share. Positive correlation between trials **inflates the true variance** above $\sum p_i(1-p_i)$, so a naive z-score overstates significance.

Nothing in this vault corrects for it. The practical consequence is that expected-versus-actual gaps should be read as **suggestive rather than tested** unless the correlation structure has been addressed.

## Relation to Other Aggregation Choices

| Approach | Handles varying $p_i$? | Gives uncertainty? |
|---|---|---|
| Count successes | — | No |
| Success *rate* | No — treats all events alike | No |
| **Poisson binomial** | **Yes** | **Yes, if independent** |
| Bootstrap over events | Yes | Yes, and can capture some dependence |

The third row is why per-event models are worth building even when the question is seasonal: the difficulty-weighting is free once the model exists, and it is exactly what a raw count discards. The same reasoning underlies [[expected-goals|xG]] as a season aggregate, and [[sample-weighting]] as a training-side correction for the same imbalance.

## See Also

- [[receiving-efficiency]] · [[pass-probability-model]] · [[probabilistic-classification]]
- [[expected-goals]] · [[uncertainty-quantification]] · [[sample-weighting]] · [[predictive-validity]]
- [[physics-based-pass-probabilities|Source Summary]]
