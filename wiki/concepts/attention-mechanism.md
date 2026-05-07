---
title: "Attention Mechanism"
type: concept
tags: [attention, deep-learning, sequence-modelling]
sources: [raw/papers/attention-is-all-you-need.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-07
updated: 2026-05-07
---

# Attention Mechanism

An attention mechanism maps a query and a set of key-value pairs to an output, computed as a weighted sum of the values where weights are derived from a compatibility function between the query and each key.

## Variants

- **Additive attention** (Bahdanau et al., 2014): Uses a learned feed-forward network to compute compatibility. Similar theoretical complexity to dot-product but slower in practice.
- **Dot-product / multiplicative attention:** Computes compatibility as $QK^T$. Faster due to optimised matrix multiplication.
- **[[scaled-dot-product-attention]]:** Scales by $1/\sqrt{d_k}$ to prevent softmax saturation at large $d_k$. Used in the [[transformer]].
- **[[multi-head-attention]]:** Runs multiple attention functions in parallel on linearly projected subspaces, then concatenates results.

## Self-Attention

Self-attention (intra-attention) relates positions within a single sequence to compute a contextual representation. In the [[transformer]], self-attention replaces recurrence entirely.

## Role in the Transformer

The [[transformer]] uses attention in three ways:
1. **Encoder self-attention:** Each position attends to all positions in the previous layer.
2. **Decoder masked self-attention:** Each position attends only to earlier positions (preserving autoregression).
3. **Encoder-decoder cross-attention:** Decoder queries attend to all encoder outputs.

## See Also

- [[scaled-dot-product-attention]]
- [[multi-head-attention]]
- [[transformer]]
