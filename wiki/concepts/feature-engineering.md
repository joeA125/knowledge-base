---
title: "Feature Engineering"
type: concept
tags: [feature-engineering, machine-learning, representation-learning, interpretability, sports-analytics]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 33%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Feature Engineering

Feature engineering is the construction of input representations for a model — deriving quantities from raw data that make the learning problem easier. In sequence and spatial domains this typically means handcrafted geometric or statistical summaries: distances, angles, durations, means, variances.

## The Standard Critique

Handcrafted features are usually defended as injecting domain knowledge. The standard objections are that they:

- **discard temporal order** — mean, variance, skewness and kurtosis are permutation-invariant, so they cannot distinguish a build-up from its reverse;
- **require domain expertise** that may not exist or may be wrong;
- **are task-specific**, so they do not transfer;
- **may introduce bias**, over-emphasising whatever the designer thought mattered.

## The Symmetric Evidence

[[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]] provide an unusually clean demonstration that handcrafted features are neither universally good nor bad — their value depends entirely on what the model's representation already captures.

Two ablations, running in opposite directions:

| Model | Manipulation | Effect |
|---|---|---|
| [[sig-model]] | **Add** Seq2Event's handcrafted geometric features | Test loss **rises** at every forecasting point (Table B7) |
| [[seq2event]] | **Remove** them, leaving raw $(x, y, T)$ | Accuracy **drops** clearly (Table B8) |

The interpretation: handcrafted geometry is a **crutch for a representation that cannot recover it**. Seq2Event's transformer over raw coordinates needs to be told about angles and distances to goal. The [[path-signature]] already encodes that geometry in its iterated integrals, so supplying it again adds redundant, noisy dimensions that actively interfere.

This generalises beyond sport into a usable heuristic: **before adding a handcrafted feature, ask whether the representation can already derive it.** If so, the feature is at best redundant and at worst harmful.

## Feature Engineering Across This Vault

The vault's models sit at very different points on this spectrum:

| Model | Approach |
|---|---|
| [[vaep]] | Heavy — action features, context features, game-state features, distances, angles, speed of play |
| [[nmstpp]] | Moderate — engineered zone geometry (distance/angle to goal, coordinate deltas) alongside learned embeddings |
| [[martingale-epv]] | Moderate — situational covariates, but spatial structure learned via [[gaussian-process]] |
| [[sig-model]] | Minimal — raw $(x, y, T)$ only; geometry implicit in the signature |
| [[bert]] / [[gpt]] | None — raw tokens; all structure learned in [[pre-train-then-fine-tune|pre-training]] |

The trajectory across deep learning generally has run from heavy engineering toward learned representations. Path signatures offer a third option: a **fixed mathematical transform** that is neither handcrafted-per-task nor learned, which is why it needs no training in the encoder yet still captures order and interaction.

## Interpretability Interactions

Feature choices affect [[interpretability]] in both directions, and not always as expected:

- Handcrafted features are individually interpretable, so a model built on them can be inspected feature by feature.
- But discretisation choices made *for* interpretability sometimes cost nothing: [[nmstpp]]'s *Juego de posición* zones match raw coordinates exactly on accuracy, while producing outputs coaches can act on.
- And signature features are individually meaningless — no single iterated integral has a football interpretation — yet the resulting model's *outputs* (predicted location and action probabilities) are directly interpretable, which is what practitioners actually need.

So interpretability of the features and interpretability of the model are separable concerns.

## See Also

- [[path-signature]]
- [[sig-model]]
- [[seq2event]]
- [[nmstpp]]
- [[representation-learning]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
