---
title: "Representation Learning"
type: concept
tags: [representation-learning, machine-learning, deep-learning, feature-engineering, entity-embedding, dimensionality-reduction, pre-training, tokenization]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/variational-lossy-autoencoders.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 55%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Representation Learning

Learning what to *feed* a model rather than hand-specifying it. The premise is that most of a model's performance is determined by how its inputs are encoded, and that encoding can itself be learned.

## The Vault's Sharpest Finding

Two football sources give opposite results on handcrafted features, and the contrast is the most useful thing here.

- [[seq2event]] **degrades without** handcrafted geometric features.
- [[sig-model]] **degrades with** them.

The reconciliation: engineered features are a **crutch for a representation that cannot recover the geometry itself.** A [[path-signature]] encodes order, curvature and interaction by construction, so adding hand-built geometry injects redundancy and noise. A fixed-window transformer over zone tokens cannot, so it needs the help.

The practical rule that follows: **before adding features, ask whether the representation could learn them.** If yes, they are noise. See [[feature-engineering]].

This sits in unresolved tension with the tracking-data line, where [[expected-value-possession-framework|Fernández et al.]] and [[vdep]] hand-engineer extensively and defend it. Their defence is that they optimise *communicability* rather than accuracy — a coach can argue with [[dynamic-pressure-lines|pressure lines]] — but neither side has tested the other's claim.

## Three Routes

| Route | Mechanism | Example here |
|---|---|---|
| **Mathematical** | A principled transform with known properties | [[path-signature]], [[non-negative-matrix-factorization\|NMF]] |
| **Architectural** | Structure that makes the right thing expressible | [[graph-neural-network\|GNN]] permutation equivariance, [[fully-convolutional-network\|FCN]] weight sharing |
| **Learned end-to-end** | Train on a proxy task, keep the internals | [[player-embedding]], [[pre-train-then-fine-tune\|pre-training]] |

The first two are underrated. [[sig-model]] beats a transformer benchmark with a plain feedforward network on signature features, and [[soccermap]]'s weight sharing is what makes [[single-pixel-supervision|learning a surface from one pixel]] possible at all. Neither is a matter of scale.

## Removing Shortcuts Improves Representations

A counterintuitive result worth generalising. [[scoutgpt]] **masks position tokens during training**, forcing the model to infer role from player identity plus surrounding events. The learned [[player-embedding|embeddings]] separate by position anyway, with tactically coherent geometry, and cross-season same-player retrieval *improves* (Top-1 9.20% vs 8.48%).

Removing the easy signal produced a better representation — the same logic as [[masked-language-model|masked language modelling]], and the same logic behind [[variational-lossy-autoencoder|VLAE]] restricting its decoder's receptive field to force global structure into the latent.

The general form: **a representation learns what it is not given for free.**

## What Counts as Good

Rarely defined explicitly, and the candidates conflict:

- **Downstream task performance** — the default, and circular if you only have one task.
- **Transfer** — does it help elsewhere? The [[large-event-model|foundation-model]] ambition.
- **Structure** — does the geometry mean something? Position separation in ScoutGPT.
- **Compression** — how much can be discarded? [[variational-lossy-autoencoder|VLAE]]'s framing.

[[interpretability]] is a fifth and often traded against the rest — though [[nmstpp]]'s *Juego de posición* zoning is a rare case where it came free, performing identically to raw coordinates while producing outputs in coaching vocabulary.

## See Also

- [[feature-engineering]] · [[player-embedding]] · [[path-signature]] · [[tokenization]]
- [[graph-neural-network]] · [[fully-convolutional-network]] · [[soccermap]] · [[single-pixel-supervision]]
- [[variational-autoencoder]] · [[variational-lossy-autoencoder]] · [[pre-train-then-fine-tune]] · [[masked-language-model]]
- [[sig-model]] · [[seq2event]] · [[scoutgpt]] · [[large-event-model]] · [[interpretability]]
- [[understanding-football-possessions-path-signatures|Sig-Model Summary]]
