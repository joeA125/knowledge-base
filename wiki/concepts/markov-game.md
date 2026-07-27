---
title: "Markov Game"
type: concept
tags: [machine-learning, markov-model, reinforcement-learning, statistics, sports-analytics, stochastic-process, policy-modelling]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 32%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
---

# Markov Game

A Markov game (also called a stochastic game) is a mathematical framework for modelling sequential decision-making by multiple agents in a shared environment. It generalises the Markov Decision Process (MDP) from one agent to many, and was introduced to the multi-agent reinforcement-learning community by Littman (1994).

## Definition

A Markov game consists of:
- A set of **states** $S$;
- A set of agents, each with an **action set** $A_i$;
- A **transition function** $T(s, a_1, \ldots, a_n, s')$ giving the probability of moving to state $s'$ when each agent takes its action;
- A **reward function** $R_i$ per agent.

The defining **Markov property** is that the next state and rewards depend only on the current state and the agents' current actions — not on the full history. This memorylessness is what makes the framework tractable.

## Relation to MDPs and Markov Chains

- A **Markov chain** models state transitions with no actions or rewards.
- A **Markov Decision Process (MDP)** adds a single agent's actions and rewards.
- A **Markov game** adds multiple (often adversarial) agents.

Soccer is naturally adversarial, making the Markov game the honest framing. In practice most sports models adopt the **single-agent MDP** instead, treating the ball carrier as the agent and folding opponents into the state — a simplification that trades away equilibrium reasoning for tractability, and which every framework in this vault makes.

## Semi-Markov Processes

A **semi-Markov** process relaxes the assumption about *timing*. In a standard continuous-time Markov chain, holding time in each state must be exponentially distributed (the only memoryless continuous distribution). A semi-Markov process allows arbitrary holding-time distributions while keeping the *embedded* sequence of visited states Markovian.

[[martingale-epv|The basketball EPV model]] assumes its coarsened possession process is marginally semi-Markov (assumption A1) precisely so that:

- the embedded state sequence forms a Markov chain, making $\mathbb{E}[X \mid C_t]$ computable by transition-matrix algebra;
- while how long a player holds the ball is left unconstrained, since real possession durations are nothing like exponential.

The known failure mode is scripted play — pre-set sequences that chain states in ways the embedded chain cannot represent.

## Estimating Value Under the Observed Policy

A distinction the sports literature makes implicitly and [[expected-value-possession-framework|Fernández, Bornn & Cervone]] make explicit: the aim is **not to find the optimal policy** $\pi$, but to estimate value under the *average policy learned from historical data*.

This inverts the usual MDP objective. Reinforcement learning treats the behaviour policy as something to improve, or to correct for via importance weighting. Here it is the estimand, and it is estimated directly — action selection probabilities and pass destination probabilities are fitted models in their own right, weighting the value of each option by how likely it is to be taken.

Two reasons, and both generalise:

- **The optimal-policy counterfactual would be unfounded.** No data contains examples of perfect play from which to learn a perfect policy. Value under observed behaviour is answerable from observed behaviour.
- **The gap is the finding.** Policy-weighted value (0.032 in the source's worked example) against best-available value (0.112) measures what ordinary decision-making leaves on the pitch — structurally the RL advantage function, repurposed as an interpretive device rather than a training signal.

See [[policy-modelling]].

## Use in Sports Analytics

- **Routley & Schulte (2015)** — Markov game model for valuing actions in ice hockey.
- **Liu & Schulte (2018)** — deep RL for context-aware player evaluation in ice hockey.
- **[[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]]** — [[martingale-epv]], semi-Markov coarsening plus continuous-resolution models.
- **[[expected-threat|xT]]** and the possession-based [[expected-possession-value]] family — transition matrices over pitch zones solved by [[value-iteration]].
- **[[evaluating-football-player-actions|Decroos et al. (2019)]]** — [[vaep]], adopting the framing while estimating state values by supervised learning.
- **[[expected-value-possession-framework|Fernández et al. (2020)]]** — explicit MDP framing with a fitted behaviour policy and a decomposed state-value function.

The shared idea: the value of a game state, and hence of the action producing it, is defined via expected future reward.

## How the Models Differ in Their Use of the Framing

None solves a full Markov game — none computes equilibrium policies. They diverge in how faithfully they implement the process model:

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Basketball EPV]] | [[expected-value-possession-framework\|Soccer EPV]] |
|---|---|---|---|---|
| State | Ball's zone | Last 3 actions | Semi-Markov coarsening + full-resolution | Full tracking snapshot |
| Estimation | [[value-iteration]] on a transition matrix | Supervised [[gradient-boosting]] | [[multiresolution-modelling\|Multiresolution]] process model | [[structured-model-decomposition\|Decomposed]] supervised components |
| Policy modelled explicitly? | Implicit in transitions | No | Implicit in transitions | **Yes — fitted separately** |
| [[martingale]] structure | No | No | **Yes** | No |
| Spatial granularity | Coarse zones | Exact action locations | Exact + 7-region coarsening | Exact + $104{\times}68$ grid |

xT builds an explicit Markov chain over an impoverished state space; VAEP takes the Markov *assumption* but not the process machinery; basketball EPV builds the process model in full, which is what buys the martingale property; soccer EPV separates the policy from the value function, which is what makes the realised-versus-available comparison computable.

The trade across the last two columns is instructive. **Modelling the policy explicitly and modelling the process faithfully turn out to be substitutes rather than complements** — the basketball model gets consistency and pays in compute; the soccer model gets an interrogable policy and pays with the martingale guarantee.

## Relation to Reinforcement Learning

Markov games are the theoretical foundation of multi-agent [[reinforcement-learning]]. The same state-value machinery that underpins [[rlhf|RLHF]] appears here — action values play a role analogous to advantage estimates, quantifying how much an action improved expected outcome.

## See Also

- [[action-valuation]] · [[expected-possession-value]] · [[policy-modelling]]
- [[expected-threat]] · [[vaep]] · [[martingale-epv]]
- [[value-iteration]] · [[multiresolution-modelling]] · [[martingale]]
- [[structured-model-decomposition]] · [[reinforcement-learning]]
- [[expected-value-possession-framework|Soccer EPV Framework Summary]]
