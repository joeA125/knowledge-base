---
title: "Neural Temporal Point Process (NTPP)"
type: concept
tags: [point-process, deep-learning, sequence-modelling, event-prediction, transformer, rnn, machine-learning]
sources: [raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Neural Temporal Point Process (NTPP)

An NTPP (reviewed by Shchur, Türkmen, Januschowski & Günnemann, 2021) combines the [[point-process]] formalism with neural networks: rather than specifying a parametric intensity function, it learns the conditional distributions of event timing and marks from data.

## The Three-Step Construction

The framework prescribes a uniform recipe:

**1. Represent each event as a feature vector.**
Embed categorical attributes (event type, location class), concatenate with continuous features (inter-event time, engineered geometry), giving $\vec{y}_i = [t_i, m_i, z_i, \dots]$.

**2. Encode the history into a fixed-size vector.**
The whole preceding sequence $(\vec{y}_1, \dots, \vec{y}_{i-1})$ is compressed into a single $\vec{h}_i$. Standard choices: [[rnn|RNN]], GRU, [[lstm|LSTM]], or a [[transformer]] encoder.

**3. Predict the next event from $\vec{h}_i$.**
Any of several parameterisations: probability density, CDF, survival function, hazard function, or cumulative hazard — or, as [[nmstpp]] does, predict the value directly and train on a regression loss.

## Why the History Encoding Step Matters

Feeding raw event sequences into a model is both ineffective (noise) and inefficient (parameter blow-up). Compressing history to a fixed-size vector is the same manoeuvre as the [[encoder-decoder-bottleneck|seq2seq bottleneck]] — and inherits the same tension, since a fixed vector must summarise an arbitrarily long history.

This is precisely where the [[attention-mechanism]] earns its place. Self-attention lets the model weight historical events by relevance rather than compressing them uniformly. In [[transformer-point-process-football-event-modelling|Yeung et al.]], attention weights over a 40-event football history sit between 0.01 and 0.06 with no systematic trend, which the authors read as evidence the window length is appropriate.

## Transformer vs Recurrent Encoders

The empirical picture from the football experiments:

| Encoder | Total loss | Training time |
|---|---|---|
| Uni-LSTM | 4.51 | 129 min |
| Transformer | 4.57 | 47 min |

The LSTM is marginally more accurate; the transformer is **2.7× faster to train**. This matches the general finding in the NTPP literature and in [[seq2event]] — recurrent encoders remain competitive on accuracy, but their sequential gradient computation makes them expensive on long sequences, exactly the tradeoff that motivated the [[transformer]] in the first place.

## Why Learned Distributions Beat Parametric Ones

Classical point processes require committing to a functional form — exponential decay for Hawkes, gamma for waiting times. NTPPs make no such commitment.

The evidence in the football case is direct: NMSTPP's predicted inter-event-time CDF matches the empirical CDF closely, **despite the model never assuming a distribution for $t$** — it simply regresses the value and lets the loss shape the implied distribution.

## Applications

Originally developed for general event-sequence domains — Du et al. (2016) on recurrent marked temporal point processes, Zuo et al. (2020) and Zhang et al. (2020) on transformer and self-attentive Hawkes processes. Sports event modelling is a comparatively recent application, with [[nmstpp]] the first to bring the framework to football.

## See Also

- [[point-process]]
- [[nmstpp]]
- [[transformer]]
- [[attention-mechanism]]
- [[encoder-decoder-bottleneck]]
- [[seq2event]]
- [[transformer-point-process-football-event-modelling|Source Summary]]
