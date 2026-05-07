---
title: "Transformer"
type: concept
tags: [transformer, architecture, deep-learning, sequence-modelling]
sources: [raw/papers/attention-is-all-you-need.md]
confidence: 0.95
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-07
updated: 2026-05-07
---

# Transformer

The Transformer is a neural network architecture introduced in [[attention-is-all-you-need|Attention Is All You Need]] (Vaswani et al., 2017). It is the first sequence transduction model built entirely on [[attention-mechanism]]s, dispensing with [[recurrence]] and [[convolution]].

## Architecture

The Transformer uses a stacked [[encoder-decoder]] structure:

- **Encoder:** $N = 6$ identical layers, each with [[multi-head-attention]] + position-wise [[feed-forward-network]], wrapped in [[residual-connections]] and [[layer-normalization]].
- **Decoder:** $N = 6$ identical layers adding masked self-attention for autoregressive generation, plus cross-attention over encoder outputs.
- **Positional information** is injected via [[positional-encoding]] (sinusoidal or learned).
- Base model dimensions: $d_{\text{model}} = 512$, $d_{ff} = 2048$, $h = 8$ heads, ~65M parameters.

## Key Advantages

1. **Parallelisable:** No sequential dependency during training (unlike RNNs).
2. **Constant path length:** Any two positions connect in $O(1)$ operations via self-attention.
3. **Scalable:** Performance improves with model size, data, and compute.

## Impact

The Transformer is the foundation of nearly all modern language models ([[bert]], [[gpt]], etc.) and has expanded into vision, speech, protein folding, and other domains.

## See Also

- [[attention-mechanism]]
- [[multi-head-attention]]
- [[scaled-dot-product-attention]]
- [[positional-encoding]]
- [[encoder-decoder]]
