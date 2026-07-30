---
title: "INLA (Integrated Nested Laplace Approximation)"
type: concept
tags: [bayesian, inference, approximation, statistics, gaussian-process, hierarchical-model, spatiotemporal]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# INLA (Integrated Nested Laplace Approximation)

INLA (Rue, Martino & Chopin, 2009) is a deterministic method for approximate Bayesian inference in *latent Gaussian models* — models where a large set of latent parameters has a Gaussian prior and observations are conditionally independent given those latents. It replaces MCMC sampling with nested Laplace approximations, typically running orders of magnitude faster.

## The Model Class

INLA applies when a model has three levels:
1. A likelihood $\prod_i p(y_i \mid x_i, \boldsymbol{\theta})$ with conditional independence.
2. A latent Gaussian field $\mathbf{x} \sim \mathcal{N}(\mathbf{0}, \boldsymbol{\Sigma}(\boldsymbol{\theta}))$ — often a Gaussian Markov random field with sparse precision.
3. A small number of hyperparameters $\boldsymbol{\theta}$.

This covers a very wide class: generalised linear mixed models, spatial and spatio-temporal models, [[gaussian-process]] regression with GMRF representations, and [[car-prior|CAR]]-structured hierarchical models.

## How It Works

Rather than sampling, INLA approximates the marginal posteriors directly. It uses a Laplace approximation for $p(\boldsymbol{\theta} \mid \mathbf{y})$, then nested Laplace approximations for each $p(x_i \mid \boldsymbol{\theta}, \mathbf{y})$, and numerically integrates the latter over a grid of $\boldsymbol{\theta}$ values. Because the latent field's precision matrix is sparse, the required linear algebra is cheap.

The "integrated" refers to marginalising over hyperparameters; the "nested" to Laplace approximations applied inside Laplace approximations.

## Use in the EPV Model

[[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] rely on INLA at three points:

| Model | Likelihood form | Scale |
|---|---|---|
| Microtransition (player movement) | Gaussian with GMRF spatial acceleration field | 461 processors, ≤18h each |
| Macrotransition entry ([[competing-risks]] hazards) | Poisson regression | ~24h per event type, 120GB RAM |
| Shot probability | Logistic regression | Smaller (shot events only) |

The macrotransition design matrix has **30.4 million rows and at least 5,993 columns**. MCMC over a model of this size with a spatially structured latent field would be prohibitive; INLA makes it feasible on a single processor per event type.

## Why It Suits This Problem

- The [[gaussian-process]] spatial effects are given a GMRF representation with a sparse precision matrix — exactly INLA's sweet spot.
- The [[car-prior]] over players is itself a GMRF, so the whole latent structure stays Gaussian.
- The hyperparameter space is low-dimensional (a handful of $\tau^2$ shrinkage terms), so the numerical integration is cheap.

## Relation to Other Approximate Inference in the Vault

INLA joins a family of deterministic alternatives to sampling:

| Method | Approximation strategy |
|---|---|
| INLA | Nested Laplace + numerical integration over hyperparameters |
| [[expectation-propagation]] | Iterative moment matching of site approximations |
| [[approximate-message-passing]] | Message passing on a [[factor-graph]] |
| [[gaussian-density-filtering]] | Single forward pass of Gaussian moment matching |

All four replace exact posteriors with tractable Gaussian-family surrogates; they differ in how the approximation is constructed and whether it is refined iteratively. See [[message-passing]] for what the last two have in common.

## Partial Likelihood Caveat

The paper factorises the full likelihood and discards one term ($L_{\text{rem}}$), fitting the rest as partial likelihoods (Cox, 1975). Under mild conditions this yields consistent, asymptotically well-behaved estimators; combined with priors on some components, the resulting inference is *partially Bayesian*.

## See Also

- [[gaussian-process]] · [[car-prior]] · [[competing-risks]] · [[survival-analysis]]
- [[expectation-propagation]] · [[message-passing]] · [[bayesian-inference]] · [[stochastic-process]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
