---
title: "Gradient Boosting"
type: concept
tags: [machine-learning, gradient-boosting, statistics, random-forest, regression, probabilistic-classification, sample-weighting, training-technique, model-selection]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

Gradient boosting builds a strong predictor as an additive sequence of weak learners (typically shallow decision trees), where each new tree is fit to the negative gradient of the loss with respect to the current ensemble.

## How It Works

Starting from an initial prediction, iteratively:
1. Compute the gradient of the loss with respect to current predictions (the "pseudo-residuals").
2. Fit a new weak learner to those residuals.
3. Add it to the ensemble, scaled by a learning rate.

Over many iterations the ensemble incrementally corrects its own mistakes. Strong track record on problems with heterogeneous features, noisy data, and complex dependencies — the profile of sports [[event-stream-data]].

## Key Implementations

- **XGBoost** (Chen & Guestrin, 2016): scalable, regularised, one-hot encoding for categoricals.
- **CatBoost** (Prokhorenkova et al., 2018): native categorical handling via ordered target statistics; ordered boosting mitigates target leakage.
- **LightGBM** (Ke et al., 2017): histogram-based splits with leaf-wise growth, substantially faster on large data. Grows deeper, more irregular trees, so more prone to overfitting on small data.

## ⚠️ Performance Is Strongly Sample-Size Dependent

The vault holds three comparisons on football data, and they **disagree** — which is the useful finding.

| Source | Data scale | Result |
|---|---|---|
| [[evaluating-football-player-actions\|VAEP]] | 8.5M actions | **CatBoost best**, XGBoost close second, beating logistic regression and [[random-forest\|Random Forest]] |
| [[epv-control-duel-skills-football\|Shelopugin]] | Season-scale event data | **LightGBM beat CatBoost** on measured accuracy |
| [[optimal-decisions-shot-taking-situations\|Yeung & Fujii]] | **2,575 shots** | **XGBoost and CatBoost were the *worst* models tested** |

The third inverts the first. On shot-block prediction (cross-entropy, lower better): MLP 0.4876, ElasticNet 0.5417, historical percentage 0.5783, **XGBoost 0.6354, CatBoost 0.7096.** Both tree ensembles lost to a constant baseline, and the authors report clear signs of overfitting.

The reconciliation is sample size, not architecture quality. Boosted ensembles are high-capacity learners that need data to constrain them; with a few thousand examples they fit sample variability rather than structure. **Reaching for gradient boosting because it won on someone else's tabular problem is not safe when your dataset is three orders of magnitude smaller.**

A related lesson from the same table: giving a small model *more features* made things worse. Handing an MLP raw coordinates for all 22 players (0.5684) performed below giving it shooter features alone (0.5545). See [[theory-based-modelling]].

## Custom Objectives

All three implementations expose the loss as a user-supplied `objective`, which matters more than it usually gets credit for. Only the gradient and Hessian are needed, so any twice-differentiable loss can be substituted.

[[epv-control-duel-skills-football|Shelopugin]] uses this for [[sample-weighting|frequency-weighted]] log-loss and squared error, dividing each instance's loss by the number of times that player appears in training:

$$\text{custom-logloss}_i = \frac{1}{|\,p_i \in D\,|}\left[y_i \log p_i + (1-y_i)\log(1-p_i)\right]$$

The purpose is to stop heavily-represented players dominating models meant to describe *situations* rather than people — see [[expected-goals]] and [[duel-skill-rating]]. The modelling choice that matters is often in the objective, not the architecture.

## Contrast with Random Forest

Both are tree ensembles, but they reduce different components of error:

| | Gradient Boosting | [[random-forest\|Random Forest]] |
|---|---|---|
| Trees built | Sequentially, on residuals | In parallel, independently |
| Ensemble reduces | **Bias** | **Variance** |
| Overfits with more trees | Yes — needs early stopping | Essentially no |
| Tuning sensitivity | High | Low |

Boosting usually wins when tuned; forests are harder to break. That second property is worth more than it looks at small sample sizes.

## Use in Football Valuation

[[vaep]] uses CatBoost for scoring and conceding probabilities, attributing its edge to categorical handling against XGBoost's one-hot encoding. Training time was the trade-off: ~100 minutes per model against ~16 for XGBoost on 8.5M actions.

[[football-performance-time-series|Mendes-Neves et al.]] went the other way, using a plain Random Forest regressor for [[intent-vs-outcome-valuation|I-VAEP/O-VAEP]] — defensible because their target is one continuous value rather than two probabilities to be differenced, removing the calibration pressure.

[[epv-control-duel-skills-football|Shelopugin]] trains **nine** boosted models for one valuation system:

| Purpose | Count | Type |
|---|---|---|
| [[expected-goals\|xG]] — open play, set pieces | 2 | Classification |
| Duel context ([[duel-skill-rating]]) | 1 | Classification |
| [[expected-possession-value\|EPV]] — open play, set pieces | 2 | Regression |
| $EPV^{avg}_{duel}$, $EPV^{ind}_{duel}$ — aerial, ground | 4 | Regression |

The split into average-player and individual-player duel models is a correctness requirement, not a convenience — using the skill-aware model to set a duellist's own baseline would penalise him for being good. See [[symmetrical-duel-valuation]].

## Why It Suits Tabular Event Data

- **Heterogeneous features** — categorical action types alongside real-valued locations.
- **[[probability-calibration|Well-calibrated probabilities]]**, essential when outputs are summed and differenced.
- **No feature scaling required**, unlike logistic regression.
- **Custom objectives are cheap**, making bias corrections practical.

All four hold *given enough data*. See the warning above.

## Relation to Other Vault Methods

Gradient boosting is the non-neural alternative to the deep learning dominant elsewhere here. That divide is contested within football: [[nmstpp]], [[sig-model]], [[eventgpt]] and [[scoutgpt]] treat event streams as *sequences* with neural architectures, while the valuation line keeps treating each action as a tabular row.

## See Also

- [[vaep]] · [[expected-possession-value]] · [[expected-goals]] · [[xsot]]
- [[random-forest]] · [[probability-calibration]] · [[probabilistic-classification]] · [[model-selection]]
- [[sample-weighting]] · [[theory-based-modelling]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[epv-control-duel-skills-football|Shelopugin Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
