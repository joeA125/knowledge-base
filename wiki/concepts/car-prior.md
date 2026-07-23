---
title: "Conditional Autoregressive (CAR) Prior"
type: concept
tags: [bayesian, hierarchical-model, statistics, spatiotemporal, inference, regularization]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 28%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Conditional Autoregressive (CAR) Prior

A CAR prior (Besag, 1974) is a hierarchical Bayesian prior that shrinks each unit's parameters toward the average of its *neighbours*, where neighbourhood is specified by an adjacency matrix $\mathbf{H}$:

$$\beta^\ell \mid \boldsymbol{\beta}^{(-\ell)}, \tau^2 \sim \mathcal{N}\left(\frac{1}{n_\ell}\sum_{k: H_{\ell k} = 1} \beta^k, \; \frac{\tau^2}{n_\ell}\right)$$

where $n_\ell$ is the number of neighbours of unit $\ell$. Units with more neighbours get tighter priors — more information is available to constrain them.

## Contrast with Exchangeable Hierarchical Priors

A standard hierarchical model shrinks every unit toward one global mean, encoding an assumption that units are exchangeable. [[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] argue this is too strong for NBA players:

> LeBron James and Steve Novak are both listed as small forwards, yet James is one of the league's most prolific short-range scorers while Novak has not scored a layup since 2012.

Shrinking Novak's parameters toward James's — or both toward a league-wide mean — would badly distort both. A CAR prior instead shrinks each player toward players who genuinely resemble him.

## Defining Similarity from Behaviour, Not Labels

The paper's neighbourhood construction is the interesting part. Rather than trusting the league's positional labels, it derives similarity from where players actually spend their time:

1. Divide the offensive half-court into 575 bins of 4 square feet.
2. Count each player's occupancy per bin, forming a $461 \times 575$ matrix $\mathbf{G}$ (square-rooted for variance stabilisation).
3. Apply [[non-negative-matrix-factorization|NMF]] to get $\mathbf{G} \approx \mathbf{U}\mathbf{V}$ with rank $r = 5$. Rows of $\mathbf{U}$ are 5-dimensional "positions" learned from data.
4. Set $H_{\ell k} = 1$ if player $k$ is among player $\ell$'s eight nearest neighbours in $\mathbf{U}$-space, then symmetrise.

LeBron James's learned neighbours: Iguodala, Barnes, Paul George, Kobe Bryant, Evan Turner, Carmelo Anthony, Stuckey, Will Barton, Rudy Gay — a more defensible peer group than "all small forwards".

## Why Shrinkage Is Necessary Here

Computing [[expected-possession-value]] requires integrating over *all possible* future paths, including situations a player never faced. DeAndre Jordan attempted no three-pointers in 2013–14, yet EPV for any possession with him on the court needs an estimate of his shooting ability from everywhere on the floor. Only borrowing strength from similar players makes such counterfactual estimates possible.

The paper's out-of-sample results bear this out: without shrinkage, the full spatial model sometimes performs *worse* than a model with no spatial effects at all — overfitting sparse per-player data. With CAR shrinkage it is consistently best.

## Prior on the Shrinkage Strength

$\tau^2 \sim \text{InvGam}(1,1)$ — deliberately informative, to avoid the degenerate endpoints. Very large $\tau^2$ means no pooling (overfitting); $\tau^2 = 0$ means complete pooling, which would erase the player-specific differences the whole model exists to capture.

## Applications Beyond Players

The same CAR structure is applied to the [[gaussian-process]] basis weights $\mathbf{w}_j^\ell$, so spatial surfaces are shrunk toward those of similar players as well — sharing information across both space and player identity simultaneously.

## See Also

- [[non-negative-matrix-factorization]]
- [[gaussian-process]]
- [[inla]]
- [[bayesian-inference]]
- [[regularization]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
