---
title: "Encoder-Decoder Bottleneck"
type: concept
tags: [deep-learning, machine-translation, sequence-modelling, architecture]
sources: [raw/papers/neural-machine-translation.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-05-08
---

# Encoder-Decoder Bottleneck

The encoder-decoder bottleneck refers to the problem that arises in [[encoder-decoder]] architectures when the encoder must compress all information from a variable-length input sequence into a single fixed-length vector $c$.

## The Problem

As input sequences grow longer, the fixed-length vector cannot retain all necessary information. Cho et al. (2014b) demonstrated that basic encoder-decoder performance degrades rapidly as sentence length increases.

## Solution: Attention

[[neural-machine-translation|Bahdanau et al. (2014)]] solved this by introducing [[additive-attention]], replacing the single fixed context vector with a dynamic, position-specific context vector $c_i = \sum_j \alpha_{ij} h_j$ computed at each decoding step. This frees the encoder from having to compress everything into one vector.

The [[transformer]] later eliminated recurrence entirely, using [[multi-head-attention]] with no fixed-length bottleneck at all.

## See Also

- [[encoder-decoder]]
- [[additive-attention]]
- [[attention-mechanism]]
- [[transformer]]
