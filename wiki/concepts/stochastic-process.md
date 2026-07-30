---
title: "Stochastic Process"
type: concept
tags: [stochastic-process, statistics, probabilistic-graphical-model, markov-model, point-process, time-series, inference]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.85
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Stochastic Process

A collection of random variables indexed by time (or space, or both) — a probability model over *whole trajectories* rather than over single outcomes.

The vault holds several members of this family, and this page exists to make the relationships between them legible, since they are usually encountered one at a time.

## The Family in This Vault

| Process | Defining property | Where used |
|---|---|---|
| [[martingale]] | $\mathbb{E}[X_{t+s} \mid \mathcal{F}_t] = X_t$ | [[martingale-epv]] — the guarantee that makes the value curve readable |
| Markov chain | Future depends only on present | [[expected-threat\|xT]]'s zone transitions; [[absorbing-markov-chain]] |
| Semi-Markov | Markov in state sequence, arbitrary holding times | [[martingale-epv]]'s coarsening |
| [[point-process]] | Random *arrivals* rather than random values | [[nmstpp]], [[football-event-sequences-point-process-mixture\|possession clustering]], [[obso\|PPCF]] |
| [[gaussian-process]] | Any finite collection is jointly Gaussian | Spatial effects in the basketball EPV model |
| Diffusion / SDE | Continuous paths, infinitesimal increments | Not held; the natural continuous-time alternative |

Two organising distinctions cut across the table.

**Discrete versus continuous time.** [[expected-threat|xT]] steps event to event; [[martingale-epv]] and [[obso|OBSO]]'s Poisson control model run in continuous time. The choice determines whether *when* something happens is a modelled quantity or an index — which is exactly the criticism [[nmstpp]] makes of its predecessors.

**Random values versus random arrivals.** A Markov chain asks "what state next?"; a [[point-process]] asks "when does the next thing happen, and what kind?" The football literature has moved from the first toward the second, which is why [[event-prediction]] models increasingly forecast timing.

## Why the Process View Buys Anything

Modelling the *process* rather than fitting a function to outcomes has consequences beyond elegance, and the vault has a clean demonstration of each.

**Guarantees about behaviour.** Because [[martingale-epv|EPV]] is a genuine conditional expectation, the tower property makes it a martingale, so its fluctuations are known to reflect events rather than estimator noise. [[multiresolution-stochastic-process-nba-possessions|Cervone et al.]] argue explicitly that regressing features onto outcomes **cannot** deliver this. Nothing else in this vault has an equivalent property — see [[interpretability]] for why that counts as a distinct kind of explainability.

**Closed-form quantities.** An [[absorbing-markov-chain]]'s fundamental matrix gives expected visits and absorption times directly; [[value-iteration]] converges to a fixed point. These are properties of the process, not fitted values.

**Composition across scales.** [[multiresolution-modelling]] combines a coarse semi-Markov chain with fine-grained continuous models, which is only coherent because both are processes over the same trajectories.

## The Cost

The basketball EPV model is the vault's cautionary example: **461 processors**, 18-hour fits, and the authors' own concession that this "likely limits EPV discussions to academic circles and professional teams with access to the appropriate resources."

The same authors later abandoned the process model for nine supervised components — gaining real-time inference and off-ball coverage, losing the martingale guarantee. See [[martingale-epv]] for that trade, which is the clearest evidence in this vault that a process model's rigour is a **cost** as well as a virtue.

## Inference

Process models rarely admit closed-form posteriors, so the vault's instances lean on approximation: [[inla]] for latent Gaussian models, [[car-prior|CAR priors]] for spatial pooling, [[expectation-maximization|EM]] for [[mixture-model|mixtures]] of point processes, and [[expectation-propagation]] for [[factor-graph|factor-graph]] formulations. See [[approximation]] and [[message-passing]].

## See Also

- [[martingale]] · [[point-process]] · [[gaussian-process]] · [[absorbing-markov-chain]] · [[competing-risks]]
- [[markov-game]] · [[multiresolution-modelling]] · [[martingale-epv]] · [[neural-temporal-point-process]]
- [[value-iteration]] · [[inla]] · [[mixture-model]] · [[event-prediction]]
- [[multiresolution-stochastic-process-nba-possessions|Basketball EPV Summary]]
