---
title: "Player Embedding"
type: concept
tags: [entity-embedding, representation-learning, sports-analytics, player-evaluation, metric-learning, dimensionality-reduction]
sources: [raw/papers/eventgpt-player-impact-from-team-action-sequences.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Player Embedding

A player embedding is a learned dense vector representing a player, trained so that the representation is useful for some predictive task. In football, embeddings support similarity search, role classification, recruitment shortlisting, and — most recently — [[counterfactual-simulation|counterfactual substitution]].

## Static vs Conditioned Embeddings

The distinction that matters for recruitment.

**Static embeddings** summarise how a player has behaved: aggregated event statistics, possession-chain summaries, passing-network position, or Decroos & Davis's *player vectors*. These answer **who a player is**.

**Conditioned embeddings**, as learned by [[eventgpt]] and [[scoutgpt]], are trained to predict what happens next *given* the player. Because the embedding must influence downstream event predictions, it encodes behavioural tendencies in a form that can be **transplanted into a different context** — which is what makes counterfactual substitution possible.

[[eventgpt-player-impact-team-action-sequences|Lee, Hong et al.]] put the limitation of the static approach directly: such representations "summarize how a player has behaved in their previous context, but they do not model how that behavior would adapt when the surrounding tactical environment changes." Static embeddings answer *who is this player*; the transfer question is *what would this player do somewhere else*.

## Position Emerges Without Supervision

Both papers report the same finding, which is the strongest evidence that these embeddings capture something real.

**Position labels are never used in training.** Yet t-SNE projections produce role-consistent clusters with tactically sensible geometry:

- Wing-backs sit *between* defenders and wingers, matching their mixed responsibilities.
- Defensive midfielders bridge the defensive cluster and the attacking one.
- Forwards, wingers and attacking midfielders separate into sub-clusters in the final third.

The structure arises purely from sequential event patterns and player-conditioned value dynamics. The embeddings encode **functional tendency rather than nominal position** — which is precisely what recruitment needs, since a label is a job title and behaviour is what transfers.

Players landing at cluster boundaries are informative: [[scoutgpt]] reports Jinsub Park between centre-back and defensive midfield, and he logged 4,383 minutes at CB, 2,081 at CDM and 1,849 at CM. Boundary positions correspond to genuine versatility rather than noise.

## Masking Improves the Representation

[[scoutgpt]] takes a further step: position tokens are **masked from the input**, so the model cannot read role off a label and must infer it from identity plus context.

This measurably improves the embeddings — cross-season same-player retrieval rises to 9.20/21.97/30.90 (Top-1/5/10) from 8.48/20.02/27.05 without masking, and beats a statistics-based baseline.

The mechanism is the same as [[masked-language-model|masked language modelling]]: removing a shortcut forces the model to use richer context, and the representation improves as a result. It is also an instance of a general pattern — **if a feature lets the model avoid learning what you want, remove it.**

## Similarity Retrieval and Its Limits

Cosine similarity in embedding space generates recruitment shortlists automatically, which [[eventgpt]] uses to retrieve candidates for Saka's role rather than hand-picking them.

But the paper's most useful result is a *negative* one. Among embedding-similar candidates, Salah (19.78) and Madueke (19.36) project high value in Saka's context while Mbeumo (11.80) and Bowen (10.10) project much lower — despite comparable similarity rank.

**Similarity is not fit.** Two players can occupy neighbouring regions of embedding space and diverge sharply once evaluated against the specific demands of a context. Embedding retrieval narrows the candidate set; counterfactual simulation is still needed to rank it.

## Relation to Other Embeddings in This Vault

- [[siamese-network]] embeddings are trained explicitly for similarity via contrastive loss. Player embeddings here are trained for *prediction* and similarity emerges as a by-product — which is why they encode behaviour rather than merely appearance.
- Word embeddings in [[gpt]] and [[bert]] are the direct analogue: entity embeddings learned as a side effect of next-token or masked-token prediction, then repurposed downstream.
- [[car-prior]] takes a non-neural route to the same problem — defining player similarity by court-occupancy [[non-negative-matrix-factorization|NMF]] rather than positional labels, for exactly the reason given here: listed position is a poor proxy for behaviour.

## See Also

- [[eventgpt]]
- [[scoutgpt]]
- [[counterfactual-simulation]]
- [[siamese-network]]
- [[car-prior]]
- [[eventgpt-player-impact-team-action-sequences|Source Summary]]
