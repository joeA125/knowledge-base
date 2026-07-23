---
title: "Seq2Event"
type: concept
tags: [sports-analytics, event-prediction, transformer, deep-learning, event-stream-data, sequence-modelling, action-valuation]
sources: [raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 40%
  ambiguous: 15%
lifecycle: draft
created: 2026-07-23
updated: 2026-07-23
---

# Seq2Event

Seq2Event ([[ian-simpson|Simpson]], Beal, Locke & Norman, KDD 2022) forecasts the **location and action type** of the next football event from a sequence of preceding events, using a [[transformer]] encoder or recurrent encoder for history representation. It also introduced **poss-util**, a possession-utilisation metric derived from the model's predictions.

> **Provenance note:** the vault's knowledge of Seq2Event comes only from its treatment as the baseline and predecessor in [[transformer-point-process-football-event-modelling|Yeung et al. (2023)]], not from the primary source. Details beyond what that paper reports are unverified.

## Contribution

Framed next-event prediction as "learning the language of soccer" — an explicit analogy between match event sequences and natural language, which motivates the use of sequence architectures developed for NLP. The model encodes a long history of events into a fixed-size vector, then uses dense layers to forecast the next event's location and type.

## poss-util

The attack probability of an event is the summed predicted probability of cross and shot. Summing across the $n$ events of a possession gives:

$$\text{poss-util} = \sum_{i=1}^{n} P(\text{Cross, Shot})$$

Possessions containing no cross or shot are multiplied by −1, then percentile ranks are applied separately to positive and negative values, yielding a metric in $[-1, 1]$.

## What NMSTPP Changed

[[nmstpp]] extends Seq2Event on two axes:

1. **Adds time.** Seq2Event forecasts location and action but not *when* the next event occurs. NMSTPP adds inter-event time as a third forecast component.
2. **Makes the components dependent.** Seq2Event predicts its outputs from a shared history vector; NMSTPP chains them via a [[point-process]] factorisation so each conditions on the previous.

Correspondingly, [[hpus]] extends poss-util by incorporating expected zone, action, *and* time, and by weighting actions within a possession rather than summing them uniformly.

In the head-to-head comparison (with Seq2Event modified to also output inter-event time for fairness), NMSTPP wins on total loss 4.40 vs 4.48–4.57.

## The Transformer-vs-LSTM Finding

Seq2Event is the origin of a result the NTPP literature repeatedly echoes: the transformer encoder is *slightly less effective but significantly more efficient* than an LSTM for encoding event history. Yeung et al. reproduce it — Uni-LSTM 4.51 total loss in 129 minutes, transformer 4.57 in 47 minutes — and cite it as their reason for choosing the transformer encoder.

This is the same efficiency argument that motivated the [[transformer]] originally: recurrent gradient computation is sequential and expensive on long sequences, whereas self-attention parallelises.

## See Also

- [[nmstpp]]
- [[hpus]]
- [[neural-temporal-point-process]]
- [[transformer]]
- [[ian-simpson]]
- [[transformer-point-process-football-event-modelling|Source Summary]]
