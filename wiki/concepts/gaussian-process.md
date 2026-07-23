---
title: "Gaussian Process"
type: concept
tags: [statistics, bayesian, gaussian-process, spatiotemporal, machine-learning, inference, approximation]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 30%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Gaussian Process

A Gaussian process (GP) is a distribution over functions such that any finite collection of function values is jointly Gaussian. It is specified by a mean function $m(\mathbf{z})$ and covariance (kernel) function $k(\mathbf{z}_1, \mathbf{z}_2)$:

$$f \sim \mathcal{GP}(m, k), \qquad \big(f(\mathbf{z}_1), \dots, f(\mathbf{z}_n)\big) \sim \mathcal{N}(\mathbf{m}, \mathbf{K})$$

GPs are the standard nonparametric Bayesian prior for unknown smooth functions — here, spatial effects over a basketball court.

## The Computational Problem

Exact GP inference costs $\mathcal{O}(n^3)$ in the number of observation locations, from inverting the $n \times n$ covariance matrix. [[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] needed spatial effects evaluated at hundreds of thousands of distinct court positions per player per event type — far beyond exact inference.

## Functional Basis Representation

The fix is to represent the GP in a fixed low-dimensional basis:

$$\xi(\mathbf{z}) = \sum_{i=1}^{d} w_i \, \phi_i(\mathbf{z}), \qquad \mathbf{w} \sim \mathcal{N}(\boldsymbol{\omega}, \boldsymbol{\Sigma})$$

This is still a GP, with mean $\Phi(\mathbf{z})'\boldsymbol{\omega}$ and covariance $\Phi(\mathbf{z}_1)'\boldsymbol{\Sigma}\Phi(\mathbf{z}_2)$, but the infinite-dimensional function is now a $d$-dimensional parameter. The paper uses $d = 10$.

Three advantages the authors emphasise:
1. **Tractability** on very large datasets.
2. **Non-stationary covariance** — the induced kernel need not depend only on $\|\mathbf{z}_1 - \mathbf{z}_2\|$, which matters because basketball court space is not spatially homogeneous (the basket and three-point line are structurally special).
3. **Composability with hierarchical priors** — a finite weight vector can be given a [[car-prior|CAR prior]] across players, which an infinite-dimensional function cannot.

## GMRF Approximation to a Matérn Kernel

The underlying bases come from a Gaussian Markov random field approximation (Lindgren et al., 2011): a triangular mesh of $d_0 = 383$ vertices over the court, with piecewise-linear interpolation between them. Choosing the weight covariance appropriately makes the induced covariance approximate a Matérn kernel:

$$\text{Cov}[x(\mathbf{z}_1), x(\mathbf{z}_2)] \approx \frac{\sigma^2}{\Gamma(\nu)2^{\nu-1}}(\kappa\|\mathbf{z}_1 - \mathbf{z}_2\|)^\nu K_\nu(\kappa\|\mathbf{z}_1 - \mathbf{z}_2\|)$$

The precision matrix is *sparse* under this construction, giving further computational savings. The 10 final bases are extracted from the 383-vertex representation by [[non-negative-matrix-factorization|NMF]], and are interpretable as shot-type motifs ranging from close-range to long-range.

## What the Spatial Effects Reveal

Because each player gets his own GP weights, the fitted surfaces are directly interpretable. LeBron James's shot-taking hazard peaks near the basket with secondary peaks at the corner threes; his pass-to-point-guard hazard peaks at the top of the arc. The acceleration fields from the microtransition model show Tony Parker attacking the basket from beyond the perimeter while Dwight Howard only accelerates to the rim from inside the paint.

This is the sharp contrast with zone-discretised models like [[expected-threat|xT]]: a GP gives a smooth surface over continuous space, where xT gives a step function over a coarse grid.

## Relation to Other Vault Concepts

- GPs are the function-space counterpart of the finite Gaussian machinery in [[trueskill]] and [[gaussian-density-filtering]] — same distributional family, extended to infinite dimensions.
- The basis-weight covariance is estimated by eigen-style decomposition, connecting to [[eigenvector]] and to the general problem of finding low-dimensional structure.
- Inference is carried out via [[inla]], which exploits the GMRF sparsity.

## See Also

- [[non-negative-matrix-factorization]]
- [[car-prior]]
- [[inla]]
- [[martingale-epv]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
