---
title: "Graph Neural Network"
type: concept
tags: [graph-neural-network, deep-learning, architecture, message-passing, representation-learning, set-modelling, trajectory-prediction]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Graph Neural Network

A network operating on graph-structured data, where entities are nodes and relationships are edges, by iteratively passing learned messages along edges.

## The Operation

One round of message passing, following Kipf et al. (2018). Given a feature vector $v_k$ for each of $K$ nodes:

$$v \to e: \quad e_{(k,j)} = f_e([v_k, v_j])$$
$$e \to v: \quad o_k = f_v\Big(\sum_{j \in N(k)} e_{(k,j)}\Big)$$

where $f_e$ and $f_v$ are small neural networks and $N(k)$ is $k$'s neighbourhood. Edge features are computed from the pair of nodes they connect, then aggregated back into each node's new state.

The aggregation is a **sum**, which is what buys the architecture its defining property.

## Permutation Equivariance

Because edge construction is symmetric between node pairs and the summation ignores edge ordering, a GNN's output is **equivariant to permutation of the nodes**: relabelling the inputs relabels the outputs identically, and nothing else changes.

This is the property that makes GNNs right for multi-agent problems. Twenty-two footballers have no canonical ordering, and a model that changes its prediction when you shuffle the roster is modelling the roster, not the game.

The vault holds two other solutions to the same problem, worth comparing:

| Approach | How ordering is handled | Example |
|---|---|---|
| **Graph structure** | Permutation-equivariant by construction | GVRNN in [[trajectory-prediction]] |
| **Sorting by a meaningful key** | Impose a canonical order | [[vdep]] — players sorted by distance to ball |
| **Attention over a set** | Learned, order-free weighting | [[read-process-write]], [[transformer]] |

VDEP's sort is the cheap version and works for tree ensembles that cannot express equivariance at all — "the nearest defender" becomes a fixed feature slot. It is lossier: the sort key must itself be meaningful, and two players at similar distances swap slots under noise.

The [[transformer]] connection runs deeper than analogy. Self-attention over a set with no positional encoding *is* message passing on a fully-connected graph with attention-weighted rather than summed aggregation — which is why [[read-process-write|Order Matters]] and the sports GNN literature keep arriving at similar constructions from different directions.

## In Multi-Agent Trajectory Models

GVRNN replaces the per-agent networks of a VRNN with GNNs at every stage — prior, encoder and decoder. Each agent's latent distribution is conditioned on all others through message passing.

The empirical payoff in [[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] is large: 0.608 m endpoint error at 4 seconds against VRNN's 5.952 m. Nearly an order of magnitude from adding relational structure to an otherwise identical model — though the comparison also confounds centralised against per-player optimisation, which the authors note.

## Caveats

- **Graph construction is a modelling choice.** Fully-connected is the default in sports, but "who interacts with whom" is itself a hypothesis; sparser graphs are cheaper and encode assumptions.
- **Over-smoothing** with depth — repeated aggregation makes node representations converge. Sports models stay shallow, which sidesteps it.
- **Cost scales with edges**, so $O(K^2)$ for a full graph. This is part of why [[c-obso]] predicts three players rather than twenty-two.

## See Also

- [[trajectory-prediction]] · [[c-obso]] · [[message-passing]] · [[factor-graph]]
- [[transformer]] · [[attention-mechanism]] · [[read-process-write]] · [[vdep]]
- [[variational-autoencoder]] · [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
