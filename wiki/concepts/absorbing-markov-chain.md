---
title: "Absorbing Markov Chain"
type: concept
tags: [markov-model, statistics, linear-algebra, stochastic-process, sports-analytics]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Absorbing Markov Chain

An absorbing Markov chain is a chain containing at least one **absorbing state** — a state that, once entered, is never left ($P(e \to e) = 1$) — reachable from every other state. The remaining states are *transient*.

This makes the chain a natural model for processes that terminate: a football possession ending in ball loss, a customer churning, a patient reaching an outcome.

## Canonical Form

Ordering transient states first, the transition matrix decomposes as

$$\boldsymbol{\Gamma} = \begin{bmatrix} \boldsymbol{Q} & \boldsymbol{R} \\ \boldsymbol{0} & \boldsymbol{I} \end{bmatrix}$$

with $\boldsymbol{Q}$ the transient-to-transient sub-stochastic block and $\boldsymbol{R}$ the transient-to-absorbing block.

## The Fundamental Matrix

Because absorption is certain, the spectral radius of $\boldsymbol{Q}$ satisfies $\rho(\boldsymbol{Q}) < 1$, so the Neumann series converges:

$$\boldsymbol{F} = \sum_{j=0}^{\infty} \boldsymbol{Q}^j = (\boldsymbol{I} - \boldsymbol{Q})^{-1}$$

$\boldsymbol{F}$ is the **fundamental matrix**, and its entries have a direct reading: $\boldsymbol{F}[e, \tilde{e}]$ is the **expected number of visits to transient state $\tilde{e}$, starting from $e$, before absorption.**

From it follow the standard quantities:

| Quantity | Formula |
|---|---|
| Expected visits to each state | $\boldsymbol{a}^\top \boldsymbol{F}$ |
| Expected steps to absorption | $\boldsymbol{a}^\top \boldsymbol{F} \boldsymbol{1}$ |
| Absorption probabilities (multiple absorbing states) | $\boldsymbol{F}\boldsymbol{R}$ |

The infinite sum over path lengths collapses into a single matrix inverse — which is what makes these quantities computable rather than merely definable.

## Application to Football Possessions

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] model event types within a possession as a finite Markov chain whose absorbing state is *End Possession*. This has a consequence worth noting: **possession length becomes a random absorption time** rather than a modelling choice. Contrast [[nmstpp]] and [[seq2event]], which impose a fixed window of past actions, and [[sig-model]], which uses whole possessions but does not model why they end.

The fundamental matrix then converts each cluster's $16 \times 16$ transition matrix into three interpretable numbers:

$$\lambda_k = 1 + \boldsymbol{a}_k^\top \boldsymbol{F}_k \boldsymbol{1}_E \quad \text{(expected events)}$$
$$\boldsymbol{\kappa}_k = \boldsymbol{a}_k^\top \boldsymbol{F}_k \quad \text{(expected visits per event type)}$$
$$\zeta_k = \boldsymbol{\mu}_k^\top [\boldsymbol{\kappa}_k, 1]^\top \quad \text{(expected duration)}$$

The last combines the chain's structure with the Gamma timing model — expected visits per event type, weighted by that type's mean inter-event time.

This is a good illustration of a general move: **report derived quantities with domain meaning, not raw parameters.** A coach cannot read a transition matrix; "this cluster averages 15 events over 33 seconds" is immediately actionable.

## Relation to Other Markov Concepts

- [[markov-game]] concerns multi-agent chains with rewards; absorption is about termination structure. [[martingale-epv|The basketball EPV model]] uses both — its coarsened chain has absorbing end-states (made 2pt, made 3pt, possession end) carrying values 2, 3, 0.
- [[value-iteration]] computes expected future *reward*; the fundamental matrix computes expected *visits* and *time*. Both are Neumann-series solutions to a linear system, one with discounting and rewards, the other with a sub-stochastic matrix.
- In [[survival-analysis]], absorption time is the event time — an absorbing chain is a discrete-state time-to-event model, connecting to [[competing-risks]] when there are several absorbing states.

## See Also

- [[markov-game]]
- [[value-iteration]]
- [[mixture-model]]
- [[competing-risks]]
- [[football-event-sequences-point-process-mixture|Source Summary]]
