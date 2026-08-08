---
title: "Multi-Agent Reinforcement Learning"
type: concept
tags: [multi-agent, reinforcement-learning, simulator, domain-adaptation, game-theory, markov-model, sports-analytics, action-valuation, off-ball, player-evaluation, trajectory-prediction, deep-learning, animal-behaviour]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 62%
  inferred: 30%
  generated: 7%
  imported: 1%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-08-07
updated: 2026-08-07
---

# Multi-Agent Reinforcement Learning

[[reinforcement-learning|RL]] where several decision-makers act in a shared environment, each with its own policy and value function — as opposed to folding the others into the environment dynamics.

The distinction matters here because **the alternative was the field's default**, and it is what confined RL valuation to the ball-holder.

## The Team-as-One-Agent Default, and What It Cost

Prior RL work in team sports — Liu & Schulte (ice hockey, 2018), Liu et al. (soccer, 2020), Routley & Schulte (2015), Rahimian et al. (2022) — treats **the team as a single agent**. One policy, one value function, actions drawn from the event stream.

Three consequences follow directly, all limitations rather than choices:

1. **Only the ball-holder can be valued**, because only the ball-holder generates an event to serve as the action.
2. **Value exists only at discrete events**, so the ~87 of 90 minutes a player spends off the ball is invisible. See [[off-ball-value]].
3. **There is no per-player attribution** without a further assumption.

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] instantiate **ten independent agents** — one per outfield attacker — each with its own $Q(s,a)$ over the same 14-action space. All three limitations lift at once.

That is the cleanest demonstration in the vault of a recurring pattern: **an analytic gap that looked like a data problem turns out to be a modelling-granularity problem.** The tracking data was there; the agent decomposition was not.

## "Independent" Is Doing a Lot of Work

Both held MARL sources use *independent* learners. Nakahara et al. say so and "omit the agent index" for simplicity; [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] "basically considered decentralized multi-agent models, which do not communicate with each other (i.e., without central control)".

| | Independent MARL (both) | Interactive MARL |
|---|---|---|
| Each agent sees | The full state, including others' positions | The full state |
| Each agent models | Others as **part of the environment** | Others as **agents with policies** |
| Non-stationarity | Unhandled | Addressed explicitly |
| Solution concept | Separate value functions | Equilibrium, or a joint policy |

Teammates *appear* in each agent's state vector, but no agent reasons about what a teammate will do. **The "multi-agent" claim is about the number of value functions, not about interaction.**

This has a concrete cost for counterfactual valuation. If a counterfactual asks "what if this player had run left instead of right", the answer is computed **holding every other player's trajectory fixed** — which is exactly what would not happen. Compare [[counterfactual-simulation]], where the regeneration route lets the world respond and pays in compounding error.

### The centralised alternative was tested, and changed nothing

> **Added 2026-08-07** on ingest of Fujii et al., and this materially weakens a natural objection.

Fujii et al. run **CDS** (Li et al., 2021), a recent *centralised* MARL method, on the football 4v8 task, and report that CDS-based results "were very similar to those in DQN-based RL models". Their conclusion:

> We confirmed that the cause of the reproducibility issue may not be the centralized/decentralized or classic/recent deep RL.

So the obvious criticism of both held papers — that their agents fail to model each other and this is what holds the approach back — **has been tested and does not survive.** Centralised learning did not help. The binding constraint is elsewhere, and the authors locate it in simulator fidelity. See [[domain-adaptation]].

That does not make the independence assumption harmless for *valuation* — a frozen-world counterfactual is still a frozen-world counterfactual. It does mean independence is not what stopped the forward approach from working.

## The Game-Theoretic Alternative

The vault holds one framework that models football agents as agents: [[optimal-decisions-shot-taking-situations|Yeung & Fujii's]] [[game-theory|game-theoretic]] shot-taking analysis, from the same group.

| | Independent MARL | Game theory (Yeung & Fujii) |
|---|---|---|
| Agents | 10 attackers, or 2–4 | 1 shooter, 1 defender |
| Opponent | In the state vector, not modelled | **Best-responding** |
| Action space | 12–14 per agent, all timesteps | 2 per agent, one decision |
| Solution | Separate $Q$-functions | **Nash equilibrium** |
| Scales to a full team | Yes | No |
| Models interaction | **No** | Yes |

Complementary in an almost embarrassing way — each has exactly what the other lacks. **Three papers now, one senior author, no cross-citation.**

## Forward and Inverse Approaches

The distinction the vault needed and lacked, stated by both RL sources and formalised by Fujii et al.:

- **Inverse** — estimate policies, rewards or values *from observed data*. Nearly everything in this vault.
- **Forward** — build a model or environment and *generate* behaviour inside it.

> **Formalised 2026-08-07.** Fujii et al. give this a precise shape. On the inverse side the transition model $\mathcal{T}^E(s'^E|s^E,\vec{a}^E)$ is **not modelled at all**, because the next state is read from the data. On the forward side a transition model $\mathcal{T}$ must be assumed. The gap between them is what [[domain-adaptation|Real-to-Sim adaptation]] exists to bridge — and is unmeasurable in principle, since measuring it would require knowing $\mathcal{T}^E$.

The vault holds one paper of each kind, from overlapping authors, and they face opposite directions:

| | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] |
|---|---|---|
| Direction | **Inverse** | **Forward** |
| Environment | None — never acts | [[nfootball\|NFootball]] |
| Deliverable | A player-valuation metric | A method |
| Alignment needed | No | **[[dynamic-time-warping\|DTW]]** — the rollout drifts out of phase |
| Outcome | A metric that disagrees with C-OBSO | **Failed to reproduce demonstrated football** |

**The forward paper is the one that fails**, and its authors rule out the algorithm as the cause. That is the strongest evidence available for why this literature is overwhelmingly inverse. See [[reinforcement-learning]].

## Where the Gains Actually Came From

Worth separating, because the papers bundle them:

| Ingredient | Enables |
|---|---|
| **Per-player agents** | Off-ball valuation at all |
| **Continuous state, discrete actions** | Valuation between events |
| **[[action-supervision]]** | Counterfactual actions having meaningful values |
| Independence assumption | Tractability — and costs interaction |

Only the first is properly "multi-agent". The second and third are orthogonal and could be applied to a single-agent formulation.

## Beyond Sport

The team-as-one-agent trap recurs wherever logged behaviour comes from a group credited jointly: trading desks, clinical teams, multi-vehicle fleets. An aggregate reward invites an aggregate agent, and the aggregate agent then cannot attribute. Decomposing into per-actor agents restores attribution at the cost of an independence assumption that is usually false.

Fujii et al.'s framing generalises further still. Their stated subject is **biological multi-agents** — the football experiment sits beside a predator-prey chase task, and the paper closes by proposing animal behaviour as the more scientifically valuable direction. See [[naoya-takeishi]] and [[yoshinobu-kawahara]].

## See Also

- [[reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[deep-q-network]] · [[action-space-design]]
- [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-reward-tradeoff]] · [[nfootball]] · [[google-research-football]]
- [[game-theory]] · [[markov-game]] · [[policy-modelling]] · [[imitation-learning]] · [[trajectory-prediction]]
- [[off-ball-value]] · [[action-valuation]] · [[counterfactual-simulation]] · [[counterfactual-baseline]]
- [[hiroshi-nakahara]] · [[keisuke-fujii]] · [[calvin-yeung]] · [[atom-scott]] · [[naoya-takeishi]] · [[yoshinobu-kawahara]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
