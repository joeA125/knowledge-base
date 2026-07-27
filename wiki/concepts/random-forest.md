---
title: "Random Forest"
type: concept
tags: [machine-learning, random-forest, regression, statistics, gradient-boosting, interpretability, feature-engineering, probabilistic-classification]
sources: [raw/papers/football-performance-time-series.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.8
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Random Forest

A random forest is an ensemble of decision trees, each grown on a bootstrap resample of the training data, with the additional constraint that **each split considers only a random subset of features**. Predictions average across trees (regression) or vote (classification).

## Why the Two Sources of Randomness

Bagging alone — bootstrap resampling — reduces variance only to the extent that the trees are decorrelated. With one dominant predictor, every bootstrapped tree splits on it first and the trees end up nearly identical, so averaging buys little.

Random **feature** subsetting at each split forces trees to use different predictors, decorrelating them and making the averaging effective. This is the specific contribution over bagged trees, and it is where the variance reduction actually comes from.

The trade-off is a small increase in individual-tree bias for a larger reduction in ensemble variance.

## Contrast with Gradient Boosting

Both are tree ensembles; they differ in what the ensemble is for.

| | Random Forest | [[gradient-boosting\|Gradient Boosting]] |
|---|---|---|
| Trees built | In parallel, independently | Sequentially, each fitting residuals |
| Individual trees | Deep, low bias, high variance | Shallow, high bias, low variance |
| Ensemble reduces | **Variance** | **Bias** |
| Overfitting with more trees | Essentially no | Yes — needs early stopping |
| Tuning sensitivity | Low | High |
| Typical accuracy | Good | Usually better, if tuned |

The practical summary: random forests are hard to break and boosting usually wins when someone has time to tune it.

## Calibration

Random forest probability estimates are **averaged vote fractions**, not calibrated probabilities, and are characteristically pulled toward the centre of the range — averaging over trees that rarely all agree makes confident predictions unlikely.

This matters for [[action-valuation]], where values are computed by **differencing** estimates at successive states. Systematic mis[[probability-calibration|calibration]] can survive the subtraction and distort action values. [[vaep|Decroos et al.]] evaluated Random Forest among their candidates and selected CatBoost partly on calibration grounds ([[probability-calibration|Brier score]] and ROC AUC).

## Use in This Vault

[[football-performance-time-series|Mendes-Neves et al.]] use a Random Forest **regressor** for their I-VAEP and O-VAEP models — `min_samples_split=50` to limit overfitting, otherwise scikit-learn defaults.

The choice is a deliberate simplification. Because their target is a continuous label on $[-1, 1]$ rather than two class probabilities, the calibration concern above is weakened: they are not composing two independently-estimated probabilities, so there is no differencing of separately-miscalibrated quantities. Regression to a signed value sidesteps the issue that pushed Decroos toward boosting.

It also serves the paper's argument. The contribution is the **[[player-rating-time-series|time-series framing]]**, not the valuation model, and a low-tuning learner keeps attention off the estimator. `min_samples_split=50` is doing real work on a dataset of hundreds of thousands of actions where unconstrained trees would fit individual events.

## Strengths and Weaknesses

**Strengths.** Minimal tuning; robust to irrelevant features; handles mixed feature types and non-linear interactions without specification; out-of-bag error estimates come free; parallelises trivially.

**Weaknesses.** Poorly calibrated probabilities; large memory footprint at prediction time; cannot extrapolate beyond the range of training targets — a structural limit for any tree ensemble, since predictions are averages of observed values; feature importance measures are biased toward high-cardinality and continuous features.

The extrapolation limit is worth noting for [[action-valuation]]: a tree ensemble can never assign a state a value more extreme than anything it saw in training, so genuinely unprecedented situations are pulled toward the observed range.

## See Also

- [[gradient-boosting]]
- [[probability-calibration]]
- [[vaep]]
- [[intent-vs-outcome-valuation]]
- [[action-valuation]]
- [[feature-engineering]]
- [[football-performance-time-series|Source Summary]]
