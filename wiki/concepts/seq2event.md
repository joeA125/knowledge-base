---
title: "Seq2Event"
type: concept
tags: [sports-analytics, event-prediction, transformer, deep-learning, event-stream-data, sequence-modelling, action-valuation, feature-engineering, tokenization]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-24
---

# Seq2Event

Seq2Event ([[ian-simpson|Simpson]], Beal, Locke & Norman, KDD 2022) forecasts the **location and action type** of the next football event from a sequence of preceding events, using a [[transformer]] or recurrent encoder for history representation. It also introduced **poss-util**, a possession-utilisation metric derived from the model's predictions.

Its framing — "learning the language of soccer" — launched the [[large-event-model|football-as-language]] line that now dominates event-sequence modelling.

> **Provenance note:** vault knowledge comes from its treatment as baseline and predecessor in three later papers, not from the primary source.

## poss-util

Attack probability per event is the summed predicted probability of cross and shot, accumulated over a possession:

$$\text{poss-util} = \sum_{i=1}^{n} P(\text{Cross, Shot})$$

Possessions containing no cross or shot are multiplied by −1, then percentile-ranked separately, yielding a metric in $[-1, 1]$.

## Three Successors, Three Different Objections

Seq2Event has been extended repeatedly, and each successor objects to something different — which makes the lineage a useful map of the design space.

### NMSTPP (2023): it ignores time
[[nmstpp]] adds inter-event time as a third forecast component and chains all three via a [[point-process]] factorisation so each conditions on the previous. [[hpus]] correspondingly extends poss-util with expected zone, action, *and* time.

### Sig-Model (2025): the fixed window is the wrong unit
[[sig-model]] accepts the outputs but rejects the input structure. A fixed window of the last $k$ actions spans possession boundaries and discards possession length; the natural unit is the possession, which requires a length-agnostic encoding ([[path-signature]]).

The supporting evidence is pointed: sweeping the window across 5, 10 and 40 past actions produces **no clear optimum**, suggesting it was never really tuned.

### ScoutGPT (2026): it cannot be conditioned on a hypothetical lineup
[[scoutgpt]] notes that Seq2Event and its descendants are built to predict *observed* continuations, and lack the entity-conditioning needed to hold context fixed while swapping one player. Without that, [[counterfactual-simulation]] of a transfer is impossible.

## Its Dependence on Handcrafted Features

A finding that generalises beyond this model. Restricted to raw $(x, y, T)$ inputs, Seq2Event degrades clearly — it **requires** handcrafted geometric features (shot angles, distances to goal) to perform well.

[[sig-model]] shows the mirror image: adding those same features to a [[path-signature]] encoder makes it *worse*. Engineered geometry is a crutch for a representation that cannot recover it. See [[feature-engineering]].

## Benchmark Results Against It

| Challenger | Outcome |
|---|---|
| [[nmstpp]] | Wins on total loss (4.40 vs 4.48–4.57) |
| [[sig-model]] | Wins on total loss, MSE, Brier, KL; **loses narrowly on CEL** |
| [[scoutgpt]] | Wins on most attributes (vs an NMSTPP-derived "LEM Transformer" baseline), largest gains on continuous variables |

The Sig-Model split is informative: Seq2Event remains marginally better at classifying the *action type* while losing clearly on *where*. Its transformer over engineered features encodes action semantics well and spatial structure less well.

## A Runtime Figure Worth Correcting

Seq2Event's originally reported runtime of roughly **45 minutes** was repeated in [[transformer-point-process-football-event-modelling|Yeung et al.]] [[understanding-football-possessions-path-signatures|Hirnschall & Bajons]] re-implemented it and report **250–688 seconds** — an order of magnitude lower — attributing the difference to improved PyTorch handling of training-sample storage, not to any change in the model.

Any comparison citing the older figure overstates the transformer's cost. A useful caution about benchmark runtimes generally: they measure an implementation at a moment in time, and propagate through citation chains long after they stop being accurate.

## The Transformer-vs-LSTM Finding

Seq2Event is the origin of a result the [[neural-temporal-point-process|NTPP]] literature repeatedly echoes: the transformer encoder is *slightly less effective but significantly more efficient* than an LSTM for encoding event history. Yeung et al. reproduce it — Uni-LSTM 4.51 total loss in 129 minutes, transformer 4.57 in 47 minutes.

## See Also

- [[large-event-model]]
- [[nmstpp]]
- [[sig-model]]
- [[scoutgpt]]
- [[hpus]] · [[lpv]]
- [[feature-engineering]]
- [[transformer]]
- [[ian-simpson]]
