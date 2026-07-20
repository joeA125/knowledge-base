---
title: "Gradient Boosting"
type: concept
tags: [machine-learning, gradient-boosting, statistics]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Gradient Boosting

Gradient boosting is an ensemble machine learning technique that builds a strong predictor as an additive sequence of weak learners (typically shallow decision trees), where each new tree is fit to the residual errors (negative gradient of the loss) of the current ensemble.

## How It Works

Starting from an initial prediction, the algorithm iteratively:
1. Computes the gradient of the loss with respect to current predictions (the "pseudo-residuals").
2. Fits a new weak learner (tree) to those residuals.
3. Adds the new learner to the ensemble, scaled by a learning rate.

Over many iterations, the ensemble incrementally corrects its own mistakes. Gradient boosting has a strong track record on problems with heterogeneous features, noisy data, and complex dependencies — exactly the profile of sports [[event-stream-data]].

## Key Implementations

- **XGBoost** (Chen & Guestrin, 2016): Scalable, regularised gradient boosting. Uses one-hot encoding for categorical features.
- **CatBoost** (Prokhorenkova et al., 2018): Handles categorical features natively via ordered target statistics, avoiding one-hot encoding. Also mitigates target leakage through ordered boosting.

## Use in VAEP

The [[evaluating-football-player-actions|VAEP paper]] uses CatBoost to estimate scoring and conceding probabilities. In an empirical comparison against Logistic Regression, Random Forest, and XGBoost, CatBoost performed best on both Brier score and ROC AUC, with XGBoost a close second. The authors attribute CatBoost's edge to its intelligent handling of categorical features (action type, body part, result) compared to XGBoost's one-hot encoding.

The trade-off is training time: CatBoost took ~100 minutes per model versus ~16 minutes for XGBoost and ~4 minutes for Logistic Regression on the same 8.5M-action training set.

## Why It Suits This Task

- **Heterogeneous features:** VAEP mixes categorical (action type, body part) and real-valued (locations, time, distances) features — a natural fit for tree ensembles.
- **[[calibration|Well-calibrated probabilities]]:** Gradient boosting can produce well-calibrated probability estimates, which is essential because VAEP sums and subtracts these probabilities to derive action values.
- **No feature scaling required:** Unlike Logistic Regression, tree-based methods are invariant to monotonic feature transformations.

## Relation to Other Vault Methods

Gradient boosting is a non-neural alternative to the deep-learning models dominant elsewhere in the vault. Where the [[transformer]] and [[lstm]] excel at sequential/unstructured data, gradient-boosted trees remain state-of-the-art for tabular, heterogeneous-feature problems like event-stream analytics.

## See Also

- [[vaep]]
- [[calibration]]
- [[probabilistic-classification]]
- [[event-stream-data]]
