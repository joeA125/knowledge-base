---
title: "Long Short-Term Memory"
type: concept
tags: [deep-learning, rnn, lstm, architecture, sequence-modelling]
sources: [raw/papers/rnn-regularisation.md, raw/papers/neural-machine-translation.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 30%
  ambiguous: 10%
lifecycle: draft
created: 2026-05-08
updated: 2026-05-08
---

# Long Short-Term Memory (LSTM)

The LSTM (Hochreiter & Schmidhuber, 1997) is a recurrent neural network architecture designed to learn long-term dependencies by maintaining explicit memory cells and using gating mechanisms to control information flow.

## Architecture

The LSTM cell uses four gates computed from the current input $h_t^{l-1}$ and previous hidden state $h_{t-1}^l$:

- **Input gate** $i$: controls what new information to store.
- **Forget gate** $f$: controls what information to discard from the cell.
- **Output gate** $o$: controls what information to output.
- **Input modulation gate** $g$: candidate values to add to the cell.

$$\begin{pmatrix} i \\ f \\ o \\ g \end{pmatrix} = \begin{pmatrix} \text{sigm} \\ \text{sigm} \\ \text{sigm} \\ \text{tanh} \end{pmatrix} T_{2n,4n} \begin{pmatrix} h_t^{l-1} \\ h_{t-1}^l \end{pmatrix}$$

$$c_t^l = f \odot c_{t-1}^l + i \odot g$$
$$h_t^l = o \odot \tanh(c_t^l)$$

## Why LSTMs Work

The cell state $c_t$ provides a highway for gradients to flow across many time steps with minimal decay, addressing the vanishing gradient problem. The gates learn when to read, write, and reset the memory.

## Relation to GRU

The [[gated-recurrent-unit]] (Cho et al., 2014) simplifies the LSTM by merging the forget and input gates into a single update gate and combining the cell and hidden states, yielding fewer parameters.

## Regularisation

Standard [[dropout]] applied to recurrent connections harms LSTMs. [[dropout-for-rnns|Zaremba et al. (2014)]] showed dropout should only be applied to non-recurrent (inter-layer) connections.

## See Also

- [[gated-recurrent-unit]]
- [[dropout-for-rnns]]
- [[bidirectional-rnn]]
- [[encoder-decoder]]
