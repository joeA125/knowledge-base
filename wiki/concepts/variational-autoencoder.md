---
title: "Variational Autoencoder"
type: concept
tags: [deep-learning, generative-model, vae, bayesian, inference]
sources: [raw/papers/variational-lossy-autoencoders.md]
confidence: 0.85
provenance:
  extracted: 40%
  inferred: 50%
  ambiguous: 10%
lifecycle: draft
created: 2026-05-10
updated: 2026-05-10
---

# Variational Autoencoder

The Variational Autoencoder (VAE; Kingma & Welling, 2013; Rezende et al., 2014) is a generative model that learns a latent representation by jointly training an encoder $q(\mathbf{z}|\mathbf{x})$ and decoder $p(\mathbf{x}|\mathbf{z})$ to maximise the variational lower bound (ELBO):

$$\mathcal{L}(\mathbf{x}) = \mathbb{E}_{q(\mathbf{z}|\mathbf{x})}[\log p(\mathbf{x}|\mathbf{z})] - D_{KL}(q(\mathbf{z}|\mathbf{x}) || p(\mathbf{z}))$$

The first term encourages reconstruction; the KL term regularises the latent code toward the prior.

## When Does a VAE Autoencode?

The [[variational-lossy-autoencoder|VLAE paper]] showed that VAEs do not always autoencode: when the decoder is powerful enough (e.g., autoregressive), it can model data without using $\mathbf{z}$, causing the latent code to be ignored.

## See Also

- [[variational-lossy-autoencoder]]
- [[autoregressive-model]]
- [[bayesian-inference]]
