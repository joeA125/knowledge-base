---
title: "Expectation-Maximization (EM)"
type: concept
tags: [expectation-maximization, statistics, inference, machine-learning, mixture-model, approximation, bayesian]
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

# Expectation-Maximization (EM)

EM (Dempster, Laird & Rubin, 1977) is an iterative algorithm for maximum-likelihood estimation when some variables are unobserved. It alternates between inferring the latent variables and optimising the parameters, and is guaranteed to increase the observed-data likelihood at every iteration.

## The Algorithm

Given observed data $\mathbf{X}$, latent variables $\mathbf{Z}$, and parameters $\boldsymbol{\theta}$:

**E-step.** Compute the expected complete-data log-likelihood under the current parameters:

$$Q(\boldsymbol{\theta} \mid \boldsymbol{\theta}^{[r]}) = \mathbb{E}_{\mathbf{Z} \mid \mathbf{X}, \boldsymbol{\theta}^{[r]}}\left[\ell_c(\boldsymbol{\theta}; \mathbf{X}, \mathbf{Z})\right]$$

**M-step.** Maximise it: $\boldsymbol{\theta}^{[r+1]} = \arg\max_{\boldsymbol{\theta}} Q(\boldsymbol{\theta} \mid \boldsymbol{\theta}^{[r]})$.

## Why It Works

The direct likelihood $\ell(\boldsymbol{\theta}; \mathbf{X}) = \ln \sum_{\mathbf{Z}} f(\mathbf{X}, \mathbf{Z}; \boldsymbol{\theta})$ contains a log-of-a-sum, which rarely has closed-form derivatives. The complete-data likelihood $\ell_c$ has the sum outside the log and typically factorises, making the M-step tractable.

The monotonicity guarantee follows from Jensen's inequality: $Q$ is a lower bound on the observed-data log-likelihood that touches it at the current parameters, so increasing $Q$ cannot decrease the likelihood. EM is therefore a *minorise-maximise* algorithm.

The bound gap is a [[kl-divergence]] between the current and true posteriors over $\mathbf{Z}$ — which is exactly the connection to variational inference: the ELBO in a [[variational-autoencoder]] is the same lower bound, with the E-step's exact posterior replaced by an approximate one because exact inference is intractable.

## Generalised EM (GEM)

The monotonicity guarantee only requires that the M-step **increase** $Q$, not maximise it. Relaxing to any improvement gives GEM (McLachlan & Krishnan, 2008), which retains the likelihood-increase property while permitting partial optimisation.

This is exactly what [[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] need. Their mixture has closed-form M-step updates for the mixing proportions and Markov transition matrices, but none for the Gamma and truncated-Gaussian parameters. Rather than running an inner optimisation to convergence, they take **a single L-BFGS-B quasi-Newton step** per GEM iteration, warm-started from the previous values — cheap, and still guaranteed to improve.

The general lesson: when part of the M-step is hard, do not abandon EM; do one step of something and keep the guarantee.

## Limitations

- **Local optima only.** The likelihood surface for [[mixture-model|mixtures]] is multimodal, so EM finds whichever mode it starts near. Multi-start is essential — 1,000 random initialisations in the football application.
- **Slow near the optimum.** Convergence is linear, with rate governed by the fraction of missing information. Quasi-Newton acceleration or Aitken extrapolation are common remedies.
- **No uncertainty quantification.** EM returns a point estimate; standard errors require additional work (e.g. Louis' method, or bootstrapping).

## Relation to Other Inference Methods in This Vault

EM is the ancestor of most of the vault's approximate-inference machinery:

| Method | Relationship to EM |
|---|---|
| **EM** | Exact E-step, maximising M-step |
| **GEM** | Exact E-step, improving M-step |
| [[variational-autoencoder\|Variational inference]] | *Approximate* E-step when the posterior is intractable; same lower bound |
| [[expectation-propagation]] | Approximates by moment matching per site rather than a global bound |
| [[gaussian-density-filtering]] | A single forward pass of moment matching, no iteration |
| [[inla]] | Laplace approximation plus numerical integration, avoiding iteration entirely |

The vault acquired EP, ADF, and INLA before EM, which inverts the historical and conceptual order — EM is the foundation the others depart from.

## Common Applications

Gaussian mixture models, hidden Markov models (where the E-step is the forward-backward algorithm), factor analysis, missing-data imputation, and item-response models. Anywhere a "if only I knew $\mathbf{Z}$, this would be easy" structure appears.

## See Also

- [[mixture-model]]
- [[variational-autoencoder]]
- [[expectation-propagation]]
- [[inla]]
- [[kl-divergence]]
- [[football-event-sequences-point-process-mixture|Source Summary]]
