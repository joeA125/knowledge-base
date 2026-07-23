---
title: "Seq2Event"
type: concept
tags: [sports-analytics, event-prediction, transformer, deep-learning, event-stream-data, sequence-modelling, action-valuation, feature-engineering]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Seq2Event

Seq2Event ([[ian-simpson|Simpson]], Beal, Locke & Norman, KDD 2022) forecasts the **location and action type** of the next football event from a sequence of preceding events, using a [[transformer]] encoder or recurrent encoder for history representation. It also introduced **poss-util**, a possession-utilisation metric derived from the model's predictions.

> **Provenance note:** the vault's knowledge of Seq2Event comes from its treatment as baseline and predecessor in [[transformer-point-process-football-event-modelling|Yeung et al. (2023)]] and [[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]], not from the primary source.

## Contribution

Framed next-event prediction as "learning the language of soccer" — an explicit analogy between match event sequences and natural language, which motivates the use of sequence architectures developed for NLP. The model encodes a long history of events into a fixed-size vector, then uses dense layers to forecast the next event's location and type.

## poss-util

The attack probability of an event is the summed predicted probability of cross and shot. Summing across the $n$ events of a possession gives:

$$\text{poss-util} = \sum_{i=1}^{n} P(\text{Cross, Shot})$$

Possessions containing no cross or shot are multiplied by −1, then percentile ranks are applied separately to positive and negative values, yielding a metric in $[-1, 1]$.

## Two Successors, Two Different Criticisms

Seq2Event has been extended twice in this vault, and the two successors object to *different* aspects of it.

### NMSTPP: it ignores time
[[nmstpp]] adds inter-event time as a third forecast component and chains the three components via a [[point-process]] factorisation so each conditions on the previous. Correspondingly [[hpus]] extends poss-util with expected zone, action, and time, plus within-possession weighting.

### Sig-Model: the fixed window is the wrong unit
[[sig-model]] accepts Seq2Event's outputs but rejects its input structure. A fixed window of the last $k$ actions spans possession boundaries and discards possession length; the natural unit is the possession itself, which requires a length-agnostic encoding ([[path-signature]]).

The supporting evidence is pointed: sweeping the window across 5, 10, and 40 past actions produces **no clear optimum**, suggesting the window size was never really tuned. And restricting Seq2Event to raw $(x, y, T)$ inputs degrades it noticeably — it *depends* on handcrafted geometric features in a way the signature model does not (see [[feature-engineering]]).

## Benchmark Results Against It

| Challenger | Outcome |
|---|---|
| [[nmstpp]] | Wins on total loss (4.40 vs 4.48–4.57) |
| [[sig-model]] | Wins on total loss, MSE, Brier, KL; **loses narrowly on CEL** |

The Sig-Model split is informative: Seq2Event remains marginally better at classifying the *action type*, while losing clearly on predicting *where* it happens. Its transformer over engineered features encodes action semantics well and spatial structure less well.

## A Runtime Figure Worth Correcting

Seq2Event's originally reported runtime of roughly **45 minutes** was repeated in [[transformer-point-process-football-event-modelling|Yeung et al.]]. [[understanding-football-possessions-path-signatures|Hirnschall & Bajons]] re-implemented it and report **250–688 seconds** — an order of magnitude lower — attributing the difference to improved PyTorch handling of training-sample storage and loading, not to any change in the model.

Any comparison citing the older figure overstates the transformer's cost. This is a useful caution about benchmark runtimes in general: they measure an implementation at a moment in time, and propagate through citation chains long after they stop being accurate.

## The Transformer-vs-LSTM Finding

Seq2Event is also the origin of a result the [[neural-temporal-point-process|NTPP]] literature repeatedly echoes: the transformer encoder is *slightly less effective but significantly more efficient* than an LSTM for encoding event history. Yeung et al. reproduce it — Uni-LSTM 4.51 total loss in 129 minutes, transformer 4.57 in 47 minutes.

## See Also

- [[nmstpp]]
- [[sig-model]]
- [[hpus]]
- [[lpv]]
- [[feature-engineering]]
- [[transformer]]
- [[ian-simpson]]
