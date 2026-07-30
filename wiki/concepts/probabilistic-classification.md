---
title: "Probabilistic Classification"
type: concept
tags: [probabilistic-classification, machine-learning, statistics, calibration, class-imbalance, evaluation, gradient-boosting, uncertainty-quantification]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/football_defence_evaluation.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.85
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Probabilistic Classification

Predicting a **probability** over classes rather than a hard label. The distinction matters whenever the output is used as a quantity — compared, differenced, thresholded, or fed into another calculation — rather than merely as a decision.

## Why the Distinction Is Load-Bearing Here

Most of this vault's football valuation frameworks are probabilistic classifiers whose outputs are then **subjected to arithmetic**. [[vaep]] is the clearest case:

$$V(a_i) = \Delta P_{scores}(a_i) - \Delta P_{concedes}(a_i)$$

Two classifiers, differenced twice. A hard-label classifier could not produce this at all, and a probabilistic one that is poorly [[probability-calibration|calibrated]] produces action values that are wrong in ways no accuracy metric would reveal.

[[vdep]] does the same with a weighting constant; [[expected-value-possession-framework|Fernández et al.]] multiply nine component probabilities together. In each case the classifier's job is not to *decide* but to *quantify*.

## The Three Properties, and Why All Three Are Needed

A classifier's output can be good or bad along three independent axes. The vault holds a worked example where each is measured and they **disagree**.

| Property | Question | Metric |
|---|---|---|
| **Discrimination** | Are positives ranked above negatives? | AUC |
| **Calibration** | Do stated probabilities match frequencies? | ECE, reliability curve |
| **Sharpness** | Does it commit, or hedge at the base rate? | F1, precision–recall |

[[football-defence-evaluation-vdep|Toda et al.]]'s comparison of VAEP's two classifiers:

| | AUC | Brier | F1 |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | **0.003** | **0.000** |

$P_{concedes}$ has the *best* Brier score in the comparison and finds **no true positives at all**. It is well-calibrated, adequately discriminating by AUC, and completely unsharp — it has learned the base rate. Any single-metric reading of this table is wrong. See [[class-imbalance-evaluation]].

The general lesson: **calibration is necessary and not sufficient.** A model predicting the base rate for every input is perfectly calibrated and useless.

## Which Models Produce Usable Probabilities

Not all classifiers do, and the differences are systematic:

- **[[gradient-boosting|Gradient-boosted trees]]** tend to be reasonably calibrated out of the box. The [[evaluating-football-player-actions|VAEP paper]] relies on this, validating CatBoost by Brier score against Logistic Regression, [[random-forest|Random Forest]] and XGBoost.
- **Logistic regression** is calibrated by construction when correctly specified — it optimises log-loss directly.
- **Modern neural networks are systematically overconfident** (Guo et al., 2017), counter to the intuition that better accuracy brings better probabilities. Post-hoc correction is usually needed; see temperature scaling on [[probability-calibration]].
- **SVMs and plain decision trees** produce scores that are not probabilities at all.

## Class Imbalance Is the Recurring Difficulty

Football's positive rates are brutal — 227 conceding events in 97,335. Under that imbalance, an unweighted loss is dominated by the majority class and gradient descent finds the degenerate solution of predicting "no" always.

Two families of remedy, and they solve different problems:

- **Reweighting the loss** — class weights, resampling, focal loss. Corrects the *objective*. [[nmstpp]] class-weights its cross-entropy terms for exactly this reason. Related but distinct from [[sample-weighting]], which corrects uneven representation across *groups* rather than across *labels*.
- **Changing the target** — predict a frequent correlate instead. Corrects the *problem*. See [[rare-event-proxy-targets]].

The second is the more radical and, on the vault's evidence, the more effective: VDEP's F1 rises from 0.000 to 0.484 not by better modelling but by predicting a different event.

## Probabilities as Uncertainty

A calibrated probability is a statement about *aleatoric* uncertainty — irreducible randomness in the outcome. It says nothing about *epistemic* uncertainty, the model's own ignorance. A classifier confidently predicting 0.5 on a genuine coin flip and one predicting 0.5 because it has never seen this input are indistinguishable from the output alone. See [[uncertainty-quantification]].

The [[glicko-rating-system|Glicko]] and [[trueskill]] rating systems are unusual in this vault for tracking the second explicitly.

## See Also

- [[probability-calibration]] · [[class-imbalance-evaluation]] · [[uncertainty-quantification]]
- [[vaep]] · [[vdep]] · [[expected-goals]] · [[rare-event-proxy-targets]]
- [[gradient-boosting]] · [[random-forest]] · [[sample-weighting]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
