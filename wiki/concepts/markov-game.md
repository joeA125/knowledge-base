---
title: "Markov Game"
type: concept
tags: [machine-learning, markov-model, reinforcement-learning, statistics, sports-analytics]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Markov Game

A Markov game (also called a stochastic game) is a mathematical framework for modelling sequential decision-making by multiple agents in a shared environment. It generalises the Markov Decision Process (MDP) from one agent to many, and was introduced to the multi-agent reinforcement-learning community by Littman (1994).

## Definition

A Markov game consists of:
- A set of **states** $S$;
- A set of agents, each with an **action set** $A_i$;
- A **transition function** $T(s, a_1, \ldots, a_n, s')$ giving the probability of moving to state $s'$ when each agent takes its action;
- A **reward function** $R_i$ per agent.

The defining **Markov property** is that the next state and rewards depend only on the current state and the agents' current actions — not on the full history of how the game reached that state. This memorylessness is what makes the framework tractable.

## Relation to MDPs and Markov Chains

- A **Markov chain** models state transitions with no actions or rewards.
- A **Markov Decision Process (MDP)** adds a single agent's actions and rewards.
- A **Markov game** adds multiple (often adversarial) agents, each choosing actions that jointly determine transitions.

Soccer is naturally adversarial — two teams with opposing objectives — making the Markov game a natural fit.

## Use in Sports Analytics

Modelling a sports match as a Markov game underpins much of the action-valuation literature. The [[evaluating-football-player-actions|VAEP paper]] situates itself within this tradition, citing several works that value player actions by modelling games as Markov games:

- Routley & Schulte (2015) — a Markov game model for valuing actions in ice hockey.
- Liu & Schulte (2018) — deep reinforcement learning for context-aware player evaluation in ice hockey.
- Cervone et al. (2014) — POINTWISE, valuing decisions in basketball from optical tracking.

The shared idea is that the value of a game state (and hence of the action that produced it) can be defined via the expected future reward — for soccer, the probability of scoring or conceding — under the Markov assumption.

## How VAEP Relates

[[vaep|VAEP]] does not solve a full Markov game (it does not compute equilibrium policies). Instead, it adopts the Markov framing's core assumption — that near-future scoring/conceding probability depends on the current game state — and estimates those probabilities directly with supervised learning. The game state is approximated by the previous three actions, a deliberately truncated (but still Markovian in spirit) representation. Unlike approaches that divide the pitch into a fixed number of zones (Routley & Schulte; Nørstebø et al.), VAEP models exact action locations.

## Relation to Reinforcement Learning

Markov games are the theoretical foundation of multi-agent [[reinforcement-learning]]. The same state-value machinery that underpins [[rlhf|RLHF]] (where a reward model scores states/trajectories) appears here — action values in VAEP play a role analogous to advantage estimates in RL, quantifying how much an action improved the agent's expected outcome.

## See Also

- [[vaep]]
- [[reinforcement-learning]]
- [[expected-goals]]
- [[evaluating-football-player-actions|Source Summary]]
