---
title: "Kullback-Leibler Divergence"
type: concept
tags: [information-theory, statistics, machine-learning, evaluation, density-estimation, bayesian]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/variational-lossy-autoencoders.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 42%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Kullback-Leibler Divergence

The KL divergence measures how much one probability distribution differs from another:

$$D_{KL}(P \parallel Q) = \sum_{x} P(x) \log \frac{P(x)}{Q(x)}$$

(or the corresponding integral for continuous distributions). It is the expected excess surprise from using $Q$ to encode data actually distributed as $P$ — an information-theoretic quantity measured in nats or bits.

## Key Properties

- **Non-negative**, and zero only when $P = Q$ almost everywhere (Gibbs' inequality).
- **Asymmetric**: $D_{KL}(P \parallel Q) \neq D_{KL}(Q \parallel P)$ in general. It is *not* a metric — no triangle inequality — which is why "KL distance" is a misnomer.
- **Unbounded**: infinite if $Q$ assigns zero probability where $P$ does not.

The asymmetry has practical consequences. Minimising $D_{KL}(P \parallel Q)$ over $Q$ is *mode-covering* — $Q$ must put mass everywhere $P$ does, or the divergence blows up. Minimising $D_{KL}(Q \parallel P)$ is *mode-seeking* — $Q$ can safely ignore regions of $P$. Variational inference typically uses the latter, which is why variational posteriors tend to be over-confident.

## Where It Appears Across This Vault

KL divergence recurs so often it is worth collecting:

| Use | Role |
|---|---|
| [[variational-autoencoder\|VAE]] ELBO | $D_{KL}(q(\mathbf{z}\mid\mathbf{x}) \parallel p(\mathbf{z}))$ regularises the latent code toward the prior |
| [[variational-lossy-autoencoder\|VLAE]] | The bits-back cost $D_{KL}(q(\mathbf{z}\mid\mathbf{x}) \parallel p(\mathbf{z}\mid\mathbf{x}))$ is exactly why powerful decoders ignore the latent code |
| [[rlhf\|RLHF]] / PPO | A KL penalty against the SFT policy prevents the RL stage drifting too far from the supervised model |
| [[expectation-propagation]] | Iterative moment matching minimises a KL divergence per site |
| Cross-entropy loss | Minimising CEL is equivalent to minimising $D_{KL}$(empirical ‖ model), since the entropy of the empirical distribution is constant |
| [[non-negative-matrix-factorization\|NMF]] | The divergence minimised in [[martingale-epv]]'s factorisation is a KL-type divergence |

The last row makes a point often missed: the ubiquitous cross-entropy loss *is* KL divergence up to a constant. Training a classifier by CEL is fitting a distribution by KL.

## As a Tactical-Plausibility Metric

[[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]] use KL divergence in an unusual evaluative role, worth recording because it generalises.

Alongside accuracy-style losses (CEL, Brier), they compute the KL divergence between the model's predicted action distribution and the **empirical zone-conditioned distribution** across 8 hand-defined pitch zones:

$$KL(\mathbb{P}, \mathbb{Q}) = \sum_{a \in \mathcal{A}} p_{z_i}(a) \log \frac{p_{z_i}(a)}{q_{z_i}(a)}$$

Their rationale: CEL and Brier reward predicting the *correct* action, while KL measures whether the model's distribution is **tactically plausible** — do more shots get predicted in the penalty area, more crosses from wide attacking zones?

These come apart. A model can be accurate on average while assigning implausible probability mass in specific regions, and a model can match the empirical distribution well while being poor at individual predictions. Reporting both catches failure modes neither would alone.

The general pattern — *use KL against a conditional empirical distribution as a structural sanity check alongside pointwise accuracy* — transfers to any domain with meaningful conditioning strata.

## Related Divergences

- **Jensen-Shannon divergence**: a symmetrised, bounded variant, $\tfrac{1}{2}D_{KL}(P\|M) + \tfrac{1}{2}D_{KL}(Q\|M)$ with $M$ the mixture. Used in GAN training analysis.
- **Wasserstein distance**: a true metric that remains finite for non-overlapping supports, motivating Wasserstein GANs.
- **Brier score**: also compares predicted to actual distributions, but is a squared-error rather than a log-based measure — see [[probability-calibration]].

## See Also

- [[variational-autoencoder]]
- [[variational-lossy-autoencoder]]
- [[rlhf]]
- [[expectation-propagation]]
- [[probability-calibration]]
- [[sig-model]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
