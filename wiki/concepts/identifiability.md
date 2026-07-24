---
title: "Identifiability"
type: concept
tags: [identifiability, statistics, inference, mixture-model, machine-learning, bayesian]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 38%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Identifiability

A model is identifiable if distinct parameter values induce distinct distributions:

$$f(\cdot; \boldsymbol{\theta}) = f(\cdot; \tilde{\boldsymbol{\theta}}) \implies \boldsymbol{\theta} = \tilde{\boldsymbol{\theta}}$$

Without it, no amount of data can pin down the parameters — different values fit equally well by construction, not by chance. Identifiability is therefore a **prerequisite for estimation to be meaningful**, not a technicality: consistency results and standard errors both presuppose it.

## Why It Is Checked Before Estimation

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] state the position plainly — establishing identifiability is "a fundamental prerequisite for ensuring the validity of parameter estimation procedure and, consequently, the reliability of the estimated partition."

For clustering the stakes are concrete. If two parameter configurations produce the same data distribution but different partitions, the recovered clusters are an artifact of where the optimiser landed rather than a property of the data.

## Label Switching in Mixtures

[[mixture-model|Finite mixtures]] are never identifiable in the strict sense: permuting component labels leaves the density unchanged. The standard statement is therefore **identifiability up to label swapping**, meaning there exists a permutation $\sigma$ with $\boldsymbol{\theta} = \sigma(\tilde{\boldsymbol{\theta}})$.

This is harmless for clustering — the partition is invariant to labelling — but matters for Bayesian estimation, where MCMC chains can switch labels mid-run, making posterior means over component parameters meaningless.

## What Makes Components Separable

The football paper's conditions are instructive because they show *which* part of a complex model does the identifying work. Three assumptions:

1. All mixing proportions strictly positive.
2. Transition matrices making the transient states a single aperiodic communication class.
3. **The Gamma (timing) parameters differ across components:** $\boldsymbol{\rho}_{k,e} \neq \boldsymbol{\rho}_{\ell,e}$ for $k \neq \ell$.

The proof strategy is to take $\Delta t \to \infty$ and exploit the differing tail behaviour of the Gamma densities to isolate one component at a time; once the timing parameters are pinned down, linear independence of the resulting density families delivers the transition matrices, then the spatial parameters.

The substantive consequence: **clusters distinguished only by their spatial or transition structure, with identical timing, would not be separable by this argument.** The temporal component is carrying the identification.

## Kinds of Non-Identifiability

- **Structural** — inherent to the parameterisation. Label switching; scale/rotation indeterminacy in factor analysis; $\alpha\beta$ appearing only as a product.
- **Practical** — identifiable in principle but poorly determined by the available data, producing flat likelihood ridges. Common in over-parameterised models.
- **Partial** — some functionals of the parameters are identified while others are not. Often the identified functionals are what you actually care about.

## Where It Bites Across This Vault

Identifiability is rarely discussed in the deep-learning sources here, and the contrast is worth noticing.

Neural networks are **massively non-identifiable** — permuting hidden units, rescaling across ReLU layers, and many other symmetries leave the function unchanged. Nobody checks, because the parameters are not the object of interest: only the induced function matters, and that *is* identified.

The distinction is what a model is for. When parameters carry meaning — cluster memberships, [[car-prior|player-specific effects]], [[trueskill|skill ratings]], [[competing-risks|cause-specific hazards]] — identifiability must be established. When only predictions matter, it can be ignored. A model whose parameters are interpreted is a model whose parameters must be identified.

## See Also

- [[mixture-model]]
- [[expectation-maximization]]
- [[bayesian-inference]]
- [[football-event-sequences-point-process-mixture|Source Summary]]
