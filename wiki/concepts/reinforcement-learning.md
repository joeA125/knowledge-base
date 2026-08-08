---
title: "Reinforcement Learning"
type: concept
tags: [reinforcement-learning, machine-learning, markov-model, dynamic-programming, temporal-difference, multi-agent, action-space, simulator, domain-adaptation, discounting, policy-modelling, imitation-learning, game-theory, action-valuation, experience-replay]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/expected_value_possession_framework.md, raw/papers/training-lm-follow-instructions-with-human-feedback.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.9
provenance:
  extracted: 38%
  inferred: 57%
  generated: 4%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Reinforcement Learning

Learning to act by interacting with an environment and receiving reward. Formally, finding a policy $\pi(a|s)$ maximising expected cumulative discounted reward in a [[markov-game|Markov decision process]].

This page exists because RL vocabulary is used throughout the vault — value functions, advantage, discounting, policy — by work that is **mostly not doing reinforcement learning.** Being clear about which parts are borrowed and which are not prevents a recurring confusion.

> **Correction history.** 2026-08-07: Nakahara et al. moved from "cited, not held" to held, and the "no simulator" argument was weakened. Later the same day, on ingest of [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]], **the weakened form was confirmed by direct evidence.** See the simulator section.

## The Core Objects

| Object | Definition | Role |
|---|---|---|
| State value $V(s)$ | Expected return from $s$ under $\pi$ | "How good is this situation?" |
| Action value $Q(s,a)$ | Expected return from taking $a$ in $s$ | "How good is this move?" |
| **Advantage** $A(s,a) = Q(s,a) - V(s)$ | How much better than average | "Was that a good decision?" |
| Discount factor $\gamma$ | Geometric weight on future reward | Convergence and impatience |

Solved by [[value-iteration|dynamic programming]] when the model is known, by [[temporal-difference-learning|temporal-difference]] or policy-gradient methods when it is not. For the neural machinery — target networks, replay buffers, double Q — see [[deep-q-network]].

## What Sports Analytics Borrows — and What It Does Not

Every [[action-valuation]] framework in this vault uses the equation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

which is structurally an **advantage**. And [[expected-possession-value|EPV]] is structurally a state-value function. The vocabulary is exact.

But **almost** none of this work is reinforcement learning, because nobody is learning a policy by interaction.

| | Reinforcement learning | Sports valuation |
|---|---|---|
| Goal | Find a better policy | **Measure** the observed one |
| Policy | The output | Fixed, or itself estimated |
| Environment | Interacted with | Only observed |
| Reward | Designed to be optimised | A label to be predicted |

The reasons are practical and recur in any domain where RL is tempting but unavailable:

- **No usable simulator.** *Revised twice below — simulators exist; what is missing is transfer.*
- **Sparse reward.** Goals are ~0.2% of events. See [[rare-event-proxy-targets]].
- **Extrapolation risk in the counterfactual.** An optimal-policy value function must evaluate actions nobody took.

So the field takes RL's *analytic* apparatus and mostly discards its *control* objective. [[expected-value-possession-framework|Fernández, Bornn & Cervone]] state this explicitly: the aim is value under the **average** policy learned from history.

## Two Frameworks That Actually Do RL — and They Face Opposite Directions

Both are [[keisuke-fujii|Fujii-group]], posted within two weeks of each other in May 2023, and the contrast between them is the most instructive thing on this page.

| | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] |
|---|---|---|
| Direction | **Inverse** — estimate values from real data | **Forward** — generate behaviour in a simulator |
| Environment | None. Never acts | **[[nfootball\|NFootball]]**, purpose-built |
| Algorithm | SARSA, on-policy | DDQN, off-policy, full stabiliser stack |
| Alignment | Contemporaneous | **[[dynamic-time-warping\|DTW]]-adaptive** |
| Goal | A player-valuation metric | Reproduce real behaviour in cyberspace |
| Outcome | A metric that disagrees with C-OBSO | **Failed to reproduce demonstrated football** |

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] train ten per-player agents with SARSA on a [[temporal-difference-learning|TD]] loss over J-League tracking data, minimising the Bellman residual directly. They clear the three objections above in three ways:

| Objection | How it is handled | Cost |
|---|---|---|
| No simulator | **Sidestepped** — learns offline from logged tracking | On-policy SARSA learns the value of *observed* play |
| Sparse reward | **Deferred** — goal ± conceding, plus terminal [[expected-possession-value\|EPV]] | Inherits whatever EPV gets wrong |
| Counterfactual extrapolation | **Regularised away** by [[action-supervision]] | Now *controlled by a hyperparameter* rather than eliminated |

[[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] take the route Nakahara et al. sidestep — and it is the one that fails. See below.

## The Optimality Gap Is Tunable

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: **strengthened 2026-08-07** — Fujii et al. measure the reward/imitation trade-off directly, though against training steps and method choice rather than $\lambda_1$. Declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

[[action-supervision]]'s weight interpolates between pure value learning and pure [[imitation-learning|imitation]]. Fujii et al. add that the position on that frontier **also moves with training time** — reward is acquired first, reproducibility afterwards at reward's expense. See [[imitation-reward-tradeoff]] and [[observed-versus-optimal-decisions]].

## Where Optimal Policy *Is* Recovered

**Correction, 2026-07-27.** An earlier revision stated flatly that this literature does not solve for optimal policy. Too strong.

[[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] do — not with RL, but with [[game-theory]]. They **shrink the strategy space until every profile is enumerable**: {Shoot, Pass} × {Block, Not Block}, with payoffs for the two unobserved profiles estimated by re-running the models with the closest defender removed. That converts extrapolation into estimation, and yields a Nash equilibrium of (Pass, Block).

**The barrier to optimal-policy analysis is the size of the action space, not the observational nature of the data.** Where the space can be coarsened to a handful of options, optimal analysis is available; the cost is that the answer concerns the coarsened game.

**Sharpened 2026-08-07.** Nakahara et al. (14 actions) and Fujii et al. (12–13 actions) supply further points on this axis, and show size is not the only thing that matters — granularity and coverage vary independently. See [[action-space-design]].

## Game Theory as the Alternative Route

| | Reinforcement learning | [[game-theory\|Game theory]] |
|---|---|---|
| Other agents | Folded into the environment | **Modelled as agents** |
| Solution concept | Optimal policy against a fixed world | **Equilibrium** against a reasoning opponent |
| Needs | Reward signal, many interactions | Payoffs for every strategy profile |
| Explains *why* | Poorly, without further analysis | By construction |

Yeung & Fujii choose game theory for the last row explicitly, noting that RL "often lacks the ability to explicitly explain why a specific decision is considered optimal without supplementary manual analysis."

**The irony holds across three papers now.** Neither Nakahara et al.'s ten agents nor Fujii et al.'s decentralised agents model each other; the vault's two "multi-agent" RL papers model interaction *less* than its two-agent game-theory paper does. Fujii et al. do test a centralised alternative (CDS, Li et al. 2021) and find it changes nothing — which is evidence that the missing interaction is not what is holding the approach back.

## The Simulator Objection, Twice Revised and Now Evidenced

**Revised 2026-08-07 (first pass).** This page previously asserted that the forward approach is closed off in football because no environment is faithful enough. [[google-research-football|GFootball]] is a counter-example to the strong form, and Nakahara et al. borrow its action vocabulary while discarding its dynamics. The accurate claim was restated as:

> The forward approach is *available*; what is unavailable is evidence that a policy learned in a simulator transfers to real players.

**Confirmed 2026-08-07 (second pass).** That was an inference from a borrowing pattern. It now has direct support.

[[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] build [[nfootball|their own simulator]] — specifically because GFootball's transitions were hard to customise and passes did not fire at intended timings — train five model variants inside it on real J-League demonstrations, and **fail to reproduce demonstrated football behaviour.** DQAAS learned to pass and shoot without moving toward goal; plain DQN moved toward goal without passing or shooting. The demonstration did both.

They then test whether the cause is algorithmic, comparing decentralised against centralised MARL and classic against recent deep RL, and conclude:

> the cause of the reproducibility issue may not be the centralized/decentralized or classic/recent deep RL

attributing it instead to **"the domain-specific modeling and reality of the simulator"**.

**A purpose-built simulator, built by domain experts to fix the incumbent's specific shortcomings, still could not support imitation of real football — and the authors rule out the algorithm as the cause.** That is the strongest available evidence for the fidelity reading, because it removes the objection that a better-suited environment would have worked.

See [[domain-adaptation]] for the Real-to-Sim framing this sits inside, and [[nfootball]] for what building your own environment costs in comparability.

## Discounting, Borrowed and Repurposed

$\gamma$ in RL encodes impatience and guarantees convergence. [[temporal-discounting|Shelopugin]] uses the identical formula for **attribution decay**. Same mathematics, different justification.

Nakahara et al. set $\gamma = 1$ — safe because reward arrives only at the terminal frame of a bounded episode, but credit is spread **flat** across possessions of up to thirty seconds. Fujii et al. state $\gamma \in (0,1]$ and **never report the value used**. Three football frameworks, one symbol, values of 0.95, 1, and unreported. See [[free-parameters-load-bearing]].

## Where the Advantage Function Reappears

[[policy-modelling|Fernández et al.'s]] realised-versus-available gap — policy-weighted EPV 0.032 against best-available 0.112 — is $Q(s,a^*) - V(s)$, computed to tell a coach what was left on the pitch. Yeung & Fujii reach a convergent conclusion by an unrelated method: passing worth 0.2456 against shooting's 0.0866 when the defender blocks.

The same algebraic shape appears in [[counterfactual-baseline|counterfactual baselines]], where the reference is a population average or a predicted behaviour. Three references, three questions, one form.

## RL Proper in This Vault

- **[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. (2023)]]** — ten independent SARSA agents on J-League tracking; on- and off-ball Q-values at every timestep. **Inverse.**
- **[[adaptive-action-supervision-multi-agent-rl|Fujii et al. (2023)]]** — DQAAS: DDQN with [[dynamic-time-warping|DTW]]-adaptive [[action-supervision]], in [[nfootball|NFootball]] and a predator-prey environment. **Forward**, and the vault's clearest negative result on simulator fidelity.
- **[[rlhf|RLHF]]** ([[training-lm-follow-instructions-with-human-feedback|InstructGPT]]) — a reward model learned from human preferences, then optimised with PPO, with a [[kl-divergence|KL]] penalty anchoring to the base model. The best-behaved instance of the [[imitation-reward-tradeoff|imitation/reward trade-off]] in the vault, because the coefficient is routinely reported.
- **MDP-based decision optimisation** — Rahimian et al. (2022, 2023), Van Roy et al. (2021). Cited, not held.
- **Deep RL for team-level valuation** — Liu & Schulte (2018), Liu et al. (2020), Routley & Schulte (2015). The **team-as-one-agent** tradition Nakahara et al. depart from. Cited, not held.
- **Inverse RL for sports** — Luo, Schulte & Poupart (2020); Rahimian & Toka (2022). Cited, not held.
- **Simulator-to-reality comparison** — **Scott, Fujii & Onishi (2022)**, *How does AI play football?* Cited, not held, and now the **highest-value acquisition target** in this area. See [[atom-scott]].

## See Also

- [[markov-game]] · [[game-theory]] · [[value-iteration]] · [[temporal-difference-learning]] · [[deep-q-network]] · [[temporal-discounting]] · [[policy-modelling]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-reward-tradeoff]]
- [[google-research-football]] · [[nfootball]] · [[imitation-learning]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[action-valuation]]
- [[rlhf]] · [[rare-event-proxy-targets]] · [[possession-risk]] · [[xsot]] · [[off-ball-value]] · [[expected-possession-value]]
- [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[training-lm-follow-instructions-with-human-feedback|InstructGPT Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
