---
title: "Gated Recurrent Unit"
type: concept
tags: [deep-learning, rnn, sequence-modelling, architecture]
sources: [raw/papers/neural-machine-translation.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-05-08
---

# Gated Recurrent Unit

The Gated Recurrent Unit (GRU; Cho et al., 2014a) is a gated recurrent neural network unit, similar to LSTM but with a simpler structure. It was used in both the encoder and decoder of the [[neural-machine-translation|Bahdanau attention model]].

## Mechanism

The hidden state update is:

$$s_i = (1 - z_i) \circ s_{i-1} + z_i \circ \tilde{s}_i$$

where:
- **Update gate** $z_i = \sigma(W_z e(y_{i-1}) + U_z s_{i-1} + C_z c_i)$ controls how much of the previous state to retain.
- **Reset gate** $r_i = \sigma(W_r e(y_{i-1}) + U_r s_{i-1} + C_r c_i)$ controls how much of the previous state feeds into the candidate.
- **Candidate** $\tilde{s}_i = \tanh(W e(y_{i-1}) + U [r_i \circ s_{i-1}] + C c_i)$.

## Relation to LSTM

GRUs combine the LSTM's forget and input gates into a single update gate, and merge the cell state and hidden state. This gives fewer parameters while maintaining the ability to learn long-term dependencies through multiplicative gating.

## See Also

- [[bidirectional-rnn]]
- [[encoder-decoder]]
- [[additive-attention]]
