---
title: "Tokenization"
type: concept
tags: [tokenization, sequence-modelling, language-modelling, transformer, encoding, feature-engineering, deep-learning]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/language_understanding_gpt.md, raw/papers/bert-bidirectional-transformers.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 38%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Tokenization

Tokenization converts raw data into a finite sequence of discrete units that a sequence model can consume. It sits between the world and the model, and the choices made here bound everything downstream — a model cannot represent a distinction its tokenizer erased.

## Text Tokenization

| Scheme | Approach | Trade-off |
|---|---|---|
| Character | One token per character | Tiny vocabulary, very long sequences |
| Word | One token per word | Short sequences, huge vocabulary, out-of-vocabulary failures |
| **Subword** | Frequent pieces merged | Balances both; no OOV, since rare words decompose |

Subword schemes dominate. [[gpt]] uses **byte-pair encoding** (40K merges), iteratively merging the most frequent adjacent pair. [[bert]] uses **WordPiece** (30K vocabulary), which merges by likelihood gain rather than raw frequency.

Both guarantee any input is representable, at the cost of splitting rare words into fragments — which is why tokenizers show characteristic weaknesses on numbers, code, and morphologically rich languages.

## Tokenizing Structured Events

Sports event data is not a sequence of atomic symbols. Each event is a **tuple** — actor, action, location, time, outcome — so applying the language recipe requires deciding how to flatten structure into a sequence.

[[scoutgpt]] flattens each event into **10 atomic tokens** in a fixed hierarchical order, with continuous variables (coordinates, time deltas) quantised into discrete bins. A 56-token **context block** encodes both lineups and match state before any events.

This creates a fixed block structure — token position within a block determines what kind of thing it is — which in turn enables:

- **[[constrained-decoding|State-dependent masking]]**, since the model knows what field it is generating and can be restricted to valid values for that field.
- **Field-specific losses**, e.g. excluding player-ID prediction while keeping everything else.
- **Deterministic entity resolution**, injecting player identity from context rather than generating it.

## The Discretisation Question

Quantising continuous values into bins is lossy, and how much to lose is a live question across this vault's sports models:

| Model | Spatial representation |
|---|---|
| [[nmstpp]] | 20 zones (*Juego de posición*) |
| [[scoutgpt]] | Coordinates binned 0–105 |
| [[sig-model]] | Continuous, via [[path-signature]] |
| [[football-event-sequences-point-process-mixture\|Mixture model]] | Continuous, truncated Gaussian |

The evidence does not point one way. NMSTPP found 20 zones performed **identically** to raw coordinates for next-action forecasting, while the mixture model retained continuous coordinates specifically to get finer spatial characterisation than three zones allowed. The right granularity depends on whether spatial precision is an input to prediction or an output to be interpreted.

Note also that discretisation is what makes the *language model* framing possible at all — a token vocabulary must be finite. Models that avoid tokenising ([[sig-model]]) also avoid the transformer.

## Why Tokenization Choices Propagate

Tokenization is easy to treat as preprocessing, but it fixes the hypothesis space:

- A distinction not encoded in tokens cannot be learned.
- Vocabulary size drives embedding-table parameters and softmax cost.
- Sequence length after tokenization drives attention cost, which is quadratic.
- Token boundaries determine what the model can attend *to* — [[attention-mechanism]] operates over tokens, so the tokenizer decides the granularity of attention.

ScoutGPT's 10-tokens-per-event scheme means a 100-event episode becomes 1,056 tokens with context — a real cost, paid for the structural control it buys.

## See Also

- [[gpt]]
- [[bert]]
- [[scoutgpt]]
- [[large-event-model]]
- [[constrained-decoding]]
- [[event-stream-data]]
