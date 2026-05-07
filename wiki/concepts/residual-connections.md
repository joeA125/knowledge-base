---
title: "Residual Connections"
type: concept
tags: [architecture, deep-learning, training-technique]
sources: [raw/papers/attention-is-all-you-need.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: draft
created: 2026-05-07
updated: 2026-05-07
---

# Residual Connections

Residual connections (He et al., 2016) add the input of a sub-layer to its output: $\text{output} = x + \text{Sublayer}(x)$. This creates a shortcut path for gradients, enabling training of much deeper networks.

## In the Transformer

The [[transformer]] wraps every sub-layer (attention and feed-forward) with a residual connection followed by [[layer-normalization]]: $\text{LayerNorm}(x + \text{Sublayer}(x))$. To make this work, all sub-layers and embedding layers produce outputs of the same dimension $d_{\text{model}} = 512$.

## See Also

- [[layer-normalization]]
- [[transformer]]
