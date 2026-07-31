---
title: "Representation Learning"
type: concept
tags: [representation-learning, machine-learning, deep-learning, feature-engineering, entity-embedding, dimensionality-reduction, pre-training, tokenization, theory-based-modelling]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/variational-lossy-autoencoders.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 38%
  generated: 15%
  imported: 2%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Representation Learning

Learning what to *feed* a model rather than hand-specifying it. The premise is that most of a model's performance is determined by how its inputs are encoded, and that encoding can itself be learned.

## The Vault's Sharpest Disagreement

Three football sources give apparently contradictory results on handcrafted features.

| Source | Finding |
|---|---|
| [[seq2event]] | **Degrades without** handcrafted geometric features |
| [[sig-model]] | **Degrades with** them |
| [[xsot\|Yeung & Fujii]] | A [[theory-based-modelling\|theory-based feature]] is essential (CEL 0.4876 vs 0.5545); **raw player coordinates are actively harmful** (0.5684, worse than omitting them) |

## The Proposed Reconciliation

> ⚠️ ^[generated: this rule is a reconciliation constructed in this vault. No source states it, none of the three addresses the others, and it has never been tested against a case it was not built to fit. It also appears on [[theory-based-modelling]] and the synthesis.]

> **Encode structure the representation cannot recover *and* the data cannot support learning. Encode nothing else.**

A [[path-signature]] recovers path geometry by construction, so adding it is redundant. A small MLP on 2,575 examples cannot recover occlusion geometry, so adding it is informative. The disagreement is about which regime you are in.

**What is wrong with it as stated.** The two clauses are not independent. "The representation cannot recover it" is a property of the *architecture*; "the data cannot support learning" is a property of *where on its learning curve* a flexible model sits. That gives a 2×2 with one cell untested and Seq2Event's cell uncertain — so the rule is supported by one clean case per clause and **no case testing them jointly**.

It predicts a locatable crossover in sample size, which a data-scaling sweep on public code would find or refute. See [[handcrafted-features-rule]].

Treat it as a working heuristic, not a finding.

## Three Routes

| Route | Mechanism | Example here |
|---|---|---|
| **Mathematical** | A principled transform with known properties | [[path-signature]], [[non-negative-matrix-factorization\|NMF]] |
| **Architectural** | Structure that makes the right thing expressible | [[graph-neural-network\|GNN]] permutation equivariance, [[fully-convolutional-network\|FCN]] weight sharing |
| **Learned end-to-end** | Train on a proxy task, keep the internals | [[player-embedding]], [[pre-train-then-fine-tune\|pre-training]] |

The first two are underrated. [[sig-model]] beats a transformer benchmark with a plain feedforward network on signature features, and [[soccermap]]'s weight sharing is what makes [[single-pixel-supervision|learning a surface from one pixel]] possible at all. Neither is a matter of scale.

## Removing Shortcuts Improves Representations

[[scoutgpt]] **masks position tokens during training**, forcing the model to infer role from player identity plus surrounding events. The learned [[player-embedding|embeddings]] separate by position anyway, with tactically coherent geometry, and cross-season same-player retrieval *improves* (Top-1 9.20% vs 8.48%).

Removing the easy signal produced a better representation — the same logic as [[masked-language-model|masked language modelling]], and behind [[variational-lossy-autoencoder|VLAE]] restricting its decoder's receptive field to force global structure into the latent.

The general form: **a representation learns what it is not given for free.**^[generated: the generalisation across these three cases is drawn here; each source states only its own version]

## What Counts as Good

Rarely defined explicitly, and the candidates conflict:

- **Downstream task performance** — the default, and circular if you only have one task.
- **Transfer** — does it help elsewhere? The [[large-event-model|foundation-model]] ambition.
- **Structure** — does the geometry mean something? Position separation in ScoutGPT.
- **Compression** — how much can be discarded? [[variational-lossy-autoencoder|VLAE]]'s framing.

[[interpretability]] is a fifth and often traded against the rest — though [[nmstpp]]'s *Juego de posición* zoning is a rare case where it came free, performing identically to raw coordinates while producing outputs in coaching vocabulary.

## See Also

- [[handcrafted-features-rule]] — the open question on the reconciliation above
- [[feature-engineering]] · [[theory-based-modelling]] · [[player-embedding]] · [[path-signature]] · [[tokenization]]
- [[graph-neural-network]] · [[fully-convolutional-network]] · [[soccermap]] · [[single-pixel-supervision]]
- [[variational-autoencoder]] · [[variational-lossy-autoencoder]] · [[pre-train-then-fine-tune]] · [[masked-language-model]]
- [[sig-model]] · [[seq2event]] · [[scoutgpt]] · [[xsot]] · [[large-event-model]] · [[interpretability]]
- [[understanding-football-possessions-path-signatures|Sig-Model Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
