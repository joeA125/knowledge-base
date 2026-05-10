---
title: "Autoregressive Model"
type: concept
tags: [deep-learning, generative-model, autoregressive-model, sequence-modelling]
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

# Autoregressive Model

An autoregressive model factorises a joint distribution using the chain rule: $p(\mathbf{x}) = \prod_i p(x_i | x_{<i})$, modelling each element conditioned on all previous elements. This decomposition is assumption-free and, with a sufficiently powerful model (e.g., RNN, PixelCNN, [[transformer]]), can represent arbitrary distributions.

## Examples

- **Language:** RNN and Transformer language models predict the next token given all previous tokens.
- **Images:** PixelRNN/PixelCNN model each pixel conditioned on previous pixels in raster-scan order.
- **Audio:** WaveNet uses causal [[dilated-convolution]]s for raw audio generation.

## Relation to VAEs

When used as a VAE decoder, autoregressive models can model data without using the latent code, causing the "information preference" problem addressed by the [[variational-lossy-autoencoder]].

## See Also

- [[variational-lossy-autoencoder]]
- [[transformer]]
- [[lstm]]
