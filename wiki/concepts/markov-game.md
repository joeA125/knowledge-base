---
title: "Markov Game"
type: concept
tags: [machine-learning, markov-model, reinforcement-learning, statistics, sports-analytics, stochastic-process]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 32%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-23
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

## Semi-Markov Processes

A **semi-Markov** process relaxes the Markov chain's assumption about *timing*. In a standard continuous-time Markov chain, the holding time in each state must be exponentially distributed (the only memoryless continuous distribution). A semi-Markov process allows arbitrary holding-time distributions while keeping the *embedded* sequence of visited states Markovian.

This distinction matters for continuous-time sports models. [[martingale-epv|The basketball EPV model]] assumes its coarsened possession process is marginally semi-Markov (its assumption A1) precisely so that:

- the embedded state sequence $C^{(0)}, C^{(1)}, \dots, C^{(K)}$ forms a Markov chain, making $\mathbb{E}[X \mid C_t]$ computable by standard transition-matrix algebra;
- while how long a player holds the ball in a given state is left unconstrained, since real possession durations are nothing like exponential.

The assumption's known failure mode is scripted play — pre-set sequences that deliberately chain states together in ways the embedded chain cannot represent.

## Use in Sports Analytics

Modelling a sports match as a Markov game underpins much of the [[action-valuation]] literature:

- **Routley & Schulte (2015)** — Markov game model for valuing actions in ice hockey, using zone-discretised state spaces.
- **Liu & Schulte (2018)** — deep reinforcement learning for context-aware player evaluation in ice hockey.
- **[[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]]** — [[martingale-epv]] for basketball, using a semi-Markov coarsening combined with continuous-resolution models.
- **[[expected-threat|xT]]** (Singh, 2019) and the wider possession-based [[expected-possession-value]] family in soccer — transition matrices over pitch zones solved by [[value-iteration]].
- **[[evaluating-football-player-actions|Decroos et al. (2019)]]** — [[vaep]] for soccer, adopting the framing while estimating state values by supervised learning.

The shared idea is that the value of a game state (and hence of the action that produced it) can be defined via expected future reward — points for basketball, scoring/conceding probability for soccer.

## How the Soccer and Basketball Models Differ in Their Use of the Framing

None solves a full Markov game — none computes equilibrium policies. But they diverge in how faithfully they implement the process model:

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|EPV]] |
|---|---|---|---|
| State | Ball's zone | Last 3 actions | Semi-Markov coarsening + full-resolution conditioning |
| Estimation | [[value-iteration]] on a transition matrix | Supervised [[gradient-boosting]] | Genuine [[multiresolution-modelling\|multiresolution]] process model |
| [[martingale]] structure | No | No | Yes |
| Spatial granularity | Coarse zones | Exact action locations | Exact locations + 7-region coarsening |

xT builds an explicit Markov chain but over an impoverished state space; VAEP takes the Markov *assumption* but not the process machinery; EPV builds the process model in full, which is what buys it the martingale property.

## Relation to Reinforcement Learning

Markov games are the theoretical foundation of multi-agent [[reinforcement-learning]]. The same state-value machinery that underpins [[rlhf|RLHF]] (where a reward model scores states/trajectories) appears here — action values play a role analogous to advantage estimates in RL, quantifying how much an action improved expected outcome.

## See Also

- [[action-valuation]]
- [[expected-possession-value]]
- [[expected-threat]]
- [[vaep]]
- [[martingale-epv]]
- [[value-iteration]]
- [[multiresolution-modelling]]
- [[martingale]]
- [[reinforcement-learning]]
