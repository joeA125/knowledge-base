---
title: "Multi-Agent Reinforcement Learning"
type: concept
tags: [multi-agent, reinforcement-learning, simulator, game-theory, markov-model, sports-analytics, action-valuation, off-ball, player-evaluation, trajectory-prediction, deep-learning]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.75
provenance:
  extracted: 58%
  inferred: 33%
  generated: 8%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Multi-Agent Reinforcement Learning

[[reinforcement-learning|RL]] where several decision-makers act in a shared environment, each with its own policy and its own value function — as opposed to folding the others into the environment dynamics.

The distinction matters here because **the alternative was the field's default**, and it is what confined RL valuation to the ball-holder.

## The Team-as-One-Agent Default, and What It Cost

Prior RL work in team sports — Liu & Schulte (ice hockey, 2018), Liu et al. (soccer, 2020), Routley & Schulte (2015), Rahimian et al. (2022) — treats **the team as a single agent**. One policy, one value function, actions drawn from the event stream.

Three consequences follow directly, and all three are limitations rather than choices:

1. **Only the ball-holder can be valued**, because only the ball-holder generates an event to serve as the action.
2. **Value exists only at discrete events**, so the ~87 of 90 minutes a player spends off the ball is invisible. See [[off-ball-value]].
3. **There is no per-player attribution** without a further assumption, since the value belongs to the team.

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] instantiate **ten independent agents** — one per outfield attacker — each with its own $Q(s,a)$ over the same 14-action space, all sharing the same 92-dimensional state. All three limitations lift at once: every player has an action at every timestep, so every player has a value at every timestep.

That is the cleanest demonstration in the vault of a recurring pattern: **an analytic gap that looked like a data problem turns out to be a modelling-granularity problem.** The tracking data was there; the agent decomposition was not.

## "Independent" Is Doing a Lot of Work

The source says plainly that it considers *independent* multi-agent RL and "omits the agent index" for simplicity. That is worth unpacking, because it is the framework's largest unstated compromise.

| | Independent MARL (used here) | Interactive MARL |
|---|---|---|
| Each agent sees | The full state, including others' positions | The full state |
| Each agent models | Others as **part of the environment** | Others as **agents with policies** |
| Non-stationarity | Unhandled — others' policies shift during training | Addressed explicitly |
| Solution concept | Ten separate value functions | Equilibrium, or a joint policy |

So the teammates *appear* in each agent's state vector, but no agent reasons about what a teammate will do. **The "multi-agent" claim is about the number of value functions, not about interaction.** Ten independent learners in a shared state space is closer to ten single-agent problems with correlated inputs than to a genuine multi-agent solution.

This has a concrete cost for the paper's own headline capability. If a counterfactual asks "what if this player had run left instead of right", the answer is computed **holding every other player's trajectory fixed** — which is exactly what would not happen. Compare [[counterfactual-simulation]], where the regeneration route lets the rest of the world respond, and pays for it in compounding error.

## The Game-Theoretic Alternative

The vault already holds one framework that models football agents as agents: [[optimal-decisions-shot-taking-situations|Yeung & Fujii's]] [[game-theory|game-theoretic]] shot-taking analysis, from the same group.

| | Independent MARL (Nakahara) | Game theory (Yeung & Fujii) |
|---|---|---|
| Agents | 10 attackers | 1 shooter, 1 defender |
| Opponent | In the state vector, not modelled | **Best-responding** |
| Action space | 14 per agent, all timesteps | 2 per agent, one decision |
| Solution | Ten $Q$-functions | **Nash equilibrium** |
| Scales to a full team | Yes | No |
| Models interaction | **No** | Yes |

The two are complementary in an almost embarrassing way — each has exactly what the other lacks. Neither cites the other despite sharing a senior author, and no held source combines them. See [[reinforcement-learning]].

## Forward and Inverse Approaches

The source draws a distinction the vault has needed and lacked:

- **Inverse** — estimate policies, rewards or values *from observed data*. Nearly everything in this vault.
- **Forward** — build a model or environment and *generate* behaviour inside it. [[google-research-football|GFootball]]; Scott, Fujii & Onishi (2022); Mendes-Neves et al.'s simulator.

[[reinforcement-learning]] asserts that the forward approach is closed off in football because there is **no simulator faithful enough to interact with**. That claim needs qualifying rather than retracting.

**Simulators exist** — [[google-research-football|GFootball]] is one, and this paper borrows its action space. What does not exist is a simulator whose dynamics match *real* football closely enough that a policy learned inside it says anything about real players. So the constraint is not availability but **fidelity and transfer**, and the paper's own behaviour shows it: it takes GFootball's *action vocabulary* and discards its *dynamics*, learning entirely from logged J-League tracking data. See [[action-space-design]].

## Where the Gains Actually Came From

Worth separating, because the paper bundles them:

| Ingredient | Enables |
|---|---|
| **Per-player agents** | Off-ball valuation at all |
| **Continuous state, discrete actions** | Valuation between events |
| **[[action-supervision]]** | Counterfactual actions having meaningful values |
| Independence assumption | Tractability — and costs interaction |

Only the first is properly "multi-agent". The second and third are orthogonal and could be applied to a single-agent formulation.

## Beyond Sport

The team-as-one-agent trap recurs wherever logged behaviour comes from a group credited jointly: trading desks, clinical teams, multi-vehicle fleets, collaborative software. The pattern is the same — an aggregate reward signal invites an aggregate agent, and the aggregate agent then cannot attribute. Decomposing into per-actor agents restores attribution at the cost of an independence assumption that is usually false.

## See Also

- [[reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[action-space-design]]
- [[game-theory]] · [[markov-game]] · [[policy-modelling]] · [[imitation-learning]] · [[trajectory-prediction]]
- [[off-ball-value]] · [[action-valuation]] · [[counterfactual-simulation]] · [[counterfactual-baseline]]
- [[google-research-football]] · [[hiroshi-nakahara]] · [[keisuke-fujii]] · [[calvin-yeung]]
- [[action-valuation-multi-agent-reinforcement-learning|Source Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
