---
title: "Gradient Boosting"
type: concept
tags: [machine-learning, gradient-boosting, statistics, random-forest, regression, probabilistic-classification, sample-weighting, training-technique]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
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
- **LightGBM** (Ke et al., 2017): Histogram-based split finding with leaf-wise (rather than level-wise) tree growth, giving substantially faster training on large datasets. Grows deeper, more irregular trees for a given leaf count, so it is more prone to overfitting on small data and is usually constrained by an explicit depth or leaf limit.

The three are close in accuracy on most tabular problems, and the choice tends to be driven by dataset size, categorical handling, and training budget rather than by predictive ceiling.

## Custom Objectives

All three implementations expose the loss function as a user-supplied `objective`, which matters more in practice than it usually gets credit for. Gradient boosting only needs the gradient and Hessian of the loss with respect to the current prediction, so any twice-differentiable loss can be substituted.

[[epv-control-duel-skills-football|Shelopugin]] uses this to implement [[sample-weighting|frequency-weighted]] variants of log-loss and squared error, dividing each instance's loss by the number of times that player appears in the training set:

$$\text{custom-logloss}_i = \frac{1}{|\,p_i \in D\,|}\left[y_i \log p_i + (1-y_i)\log(1-p_i)\right]$$

The purpose is to stop heavily-represented players dominating the fit of models that are meant to describe *situations* rather than people — see [[expected-goals]] and [[duel-skill-rating]]. This is a good illustration of the general point: the modelling choice that matters is often in the objective, not the architecture.

## Contrast with Random Forest

Both are tree ensembles, but they reduce different components of error — see [[random-forest]] for the full comparison.

| | Gradient Boosting | [[random-forest\|Random Forest]] |
|---|---|---|
| Trees built | Sequentially, on residuals | In parallel, independently |
| Ensemble reduces | **Bias** | **Variance** |
| Individual trees | Shallow, high bias | Deep, high variance |
| Overfits with more trees | Yes — needs early stopping | Essentially no |
| Tuning sensitivity | High | Low |

The short version: boosting usually wins when tuned; forests are harder to break.

## Use in VAEP

The [[evaluating-football-player-actions|VAEP paper]] uses CatBoost to estimate scoring and conceding probabilities. In an empirical comparison against Logistic Regression, [[random-forest|Random Forest]], and XGBoost, CatBoost performed best on both Brier score and ROC AUC, with XGBoost a close second. The authors attribute CatBoost's edge to its intelligent handling of categorical features (action type, body part, result) compared to XGBoost's one-hot encoding.

The trade-off is training time: CatBoost took ~100 minutes per model versus ~16 minutes for XGBoost and ~4 minutes for Logistic Regression on the same 8.5M-action training set.

**A later variant went the other way.** [[football-performance-time-series|Mendes-Neves et al.]] use a plain Random Forest regressor for their [[intent-vs-outcome-valuation|I-VAEP/O-VAEP]] models. The choice is defensible because their target is a single continuous value rather than two probabilities to be differenced — which removes the calibration pressure that pushed Decroos toward boosting. See [[random-forest]].

## Use in Possession-Value Modelling

Shelopugin trains **nine** boosted models for one valuation system, which is a useful illustration of how these pipelines actually decompose:

| Purpose | Count | Type |
|---|---|---|
| [[expected-goals\|xG]] — open play, set pieces | 2 | Classification |
| Duel context (advantage term for [[duel-skill-rating]]) | 1 | Classification |
| [[expected-possession-value\|EPV]] — open play, set pieces | 2 | Regression |
| $EPV^{avg}_{duel}$ — aerial, ground | 2 | Regression |
| $EPV^{ind}_{duel}$ — aerial, ground | 2 | Regression |

LightGBM is used throughout, chosen over CatBoost on measured accuracy for these particular targets — the opposite result to the VAEP paper's comparison. Taken together, the two findings suggest the boosting-library choice is genuinely dataset-dependent rather than settled, and worth testing rather than inheriting.

The split into average-player and individual-player duel models is not a modelling convenience but a correctness requirement: using the skill-aware model to set a duellist's own baseline would penalise him for being good. See [[symmetrical-duel-valuation]].

## Why It Suits This Task

- **Heterogeneous features:** VAEP mixes categorical (action type, body part) and real-valued (locations, time, distances) features — a natural fit for tree ensembles.
- **[[probability-calibration|Well-calibrated probabilities]]:** Gradient boosting can produce well-calibrated probability estimates, which is essential because VAEP sums and subtracts these probabilities to derive action values.
- **No feature scaling required:** Unlike Logistic Regression, tree-based methods are invariant to monotonic feature transformations.
- **Custom objectives are cheap**, which makes bias corrections like frequency weighting practical to apply.

## Relation to Other Vault Methods

Gradient boosting is a non-neural alternative to the deep-learning models dominant elsewhere in the vault. Where the [[transformer]] and [[lstm]] excel at sequential/unstructured data, gradient-boosted trees remain state-of-the-art for tabular, heterogeneous-feature problems like event-stream analytics.

That divide is now contested within football specifically. [[nmstpp]], [[sig-model]], [[eventgpt]] and [[scoutgpt]] all model event streams as *sequences* with neural architectures, while the valuation line ([[vaep]], Shelopugin) keeps treating each action as a tabular row. The two families are converging on the same data from opposite directions.

## See Also

- [[vaep]] · [[expected-possession-value]] · [[expected-goals]]
- [[random-forest]] · [[probability-calibration]] · [[probabilistic-classification]]
- [[sample-weighting]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[event-stream-data]] · [[intent-vs-outcome-valuation]]
- [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
