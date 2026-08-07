---
title: "Reinforcement Learning"
type: concept
tags: [reinforcement-learning, machine-learning, markov-model, dynamic-programming, temporal-difference, multi-agent, action-space, simulator, discounting, policy-modelling, imitation-learning, game-theory, action-valuation]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/expected_value_possession_framework.md, raw/papers/training-lm-follow-instructions-with-human-feedback.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.85
provenance:
  extracted: 34%
  inferred: 61%
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

> **Correction, 2026-08-07.** This page previously listed Nakahara et al. (2023) under "cited, not held". **It is now held** — see [[action-valuation-multi-agent-reinforcement-learning]] — and it is the vault's first source where a Bellman residual is the actual training objective. Two claims below have been revised as a result: the "no simulator" argument and the scope of the "mostly not RL" framing.

## The Core Objects

| Object | Definition | Role |
|---|---|---|
| State value $V(s)$ | Expected return from $s$ under $\pi$ | "How good is this situation?" |
| Action value $Q(s,a)$ | Expected return from taking $a$ in $s$ | "How good is this move?" |
| **Advantage** $A(s,a) = Q(s,a) - V(s)$ | How much better than average | "Was that a good decision?" |
| Discount factor $\gamma$ | Geometric weight on future reward | Convergence and impatience |

Solved by [[value-iteration|dynamic programming]] when the model is known, by [[temporal-difference-learning|temporal-difference]] or policy-gradient methods when it is not.

## What Sports Analytics Borrows — and What It Does Not

Every [[action-valuation]] framework in this vault uses the equation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

which is structurally an **advantage**. And [[expected-possession-value|EPV]] is structurally a state-value function. The vocabulary is exact.

But **almost** none of this work is reinforcement learning, because nobody is learning a policy by interaction. "Almost" now does real work — see the next section.

| | Reinforcement learning | Sports valuation |
|---|---|---|
| Goal | Find a better policy | **Measure** the observed one |
| Policy | The output | Fixed, or itself estimated |
| Environment | Interacted with | Only observed |
| Reward | Designed to be optimised | A label to be predicted |

The reasons are practical and recur in any domain where RL is tempting but unavailable:

- **No usable simulator.** *Revised below* — simulators exist; what is missing is transfer.
- **Sparse reward.** Goals are ~0.2% of events, too little signal to optimise against. See [[rare-event-proxy-targets]].
- **Extrapolation risk in the counterfactual.** An optimal-policy value function must evaluate actions nobody took. Over a large action space that is extrapolation, not estimation.

So the field takes RL's *analytic* apparatus (value functions, advantage, discounting) and mostly discards its *control* objective. [[expected-value-possession-framework|Fernández, Bornn & Cervone]] state this explicitly: the aim is value under the **average** policy learned from history.

## The One Framework That Actually Does RL

> **Added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]].

[[action-valuation-multi-agent-reinforcement-learning|Nakahara, Tsutsui, Takeda & Fujii]] train **ten per-player agents** with SARSA on a [[temporal-difference-learning|TD]] loss over J-League tracking data. The Bellman residual is minimised directly. This is RL, not RL vocabulary.

It clears the three objections in three different ways, and the manner of each is instructive:

| Objection | How it is handled | Cost |
|---|---|---|
| No simulator | **Sidestepped** — learns offline from logged tracking, never interacts | On-policy SARSA, so it learns the value of *observed* play, not optimal play |
| Sparse reward | **Not solved, deferred** — goal ± conceding, supplemented by [[expected-possession-value\|EPV]] at the terminal frame | Inherits whatever EPV gets wrong |
| Counterfactual extrapolation | **Regularised away** by [[action-supervision]] | The extrapolation is now *controlled by a hyperparameter* rather than eliminated |

The second row is the quiet one. Using an existing valuation model as the **reward function** of a new one is unusual, and it makes the Q-values dependent on EPV's correctness in a way none of the group's other work is.

The third row is the important one — see below.

## The Optimality Gap Turns Out To Be Tunable

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: drawn from Nakahara et al.'s own description of the $\lambda_1$ extremes; they do not state the general claim. Declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff]

[[action-supervision]]'s weight $\lambda_1$ interpolates between pure value learning ($\lambda_1 \to 0$: counterfactuals unconstrained, players look arbitrary) and pure [[imitation-learning|imitation]] ($\lambda_1 \to \infty$: model reproduces observed choices, players look optimal by construction).

This bears directly on [[observed-versus-optimal-decisions]], which asks whether players really decide badly. A third possibility now sits alongside the two objections recorded there: **in this family of methods the size of the gap is a modelling choice, and nobody has reported the curve.**

## Where Optimal Policy *Is* Recovered

**Correction, 2026-07-27.** An earlier revision of this page stated flatly that this literature does not solve for optimal policy. That is too strong.

[[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] do — not with RL, but with [[game-theory]]. The route around the third objection above is to **shrink the strategy space until every profile is enumerable**: the shooter chooses {Shoot, Pass}, the defender {Block, Not Block}, and payoffs for the two unobserved profiles are estimated by re-running the models with the closest defender removed.

That converts extrapolation into estimation, and yields a Nash equilibrium of (Pass, Block).

The general lesson is worth separating from the football: **the barrier to optimal-policy analysis is the size of the action space, not the observational nature of the data.** Where the space can be coarsened to a handful of options, optimal analysis is available; the cost is that the answer concerns the coarsened game. "Pass" collapsing ten possible recipients into one option is exactly that cost.

**Sharpened 2026-08-07.** Nakahara et al. supply a third point on this axis — 14 actions per agent, between Yeung & Fujii's four profiles and Fernández et al.'s continuous surface — and it shows size is not the only thing that matters. Granularity and coverage vary independently, and no held framework has all three. See [[action-space-design]], where the claim is restated as `action-space-sets-the-question`.

## Game Theory as the Alternative Route

The two are often conflated because both concern optimal action:

| | Reinforcement learning | [[game-theory\|Game theory]] |
|---|---|---|
| Other agents | Folded into the environment | **Modelled as agents** |
| Solution concept | Optimal policy against a fixed world | **Equilibrium** against a reasoning opponent |
| Needs | Reward signal, many interactions | Payoffs for every strategy profile |
| Explains *why* | Poorly, without further analysis | By construction — the payoff table is the explanation |

Yeung & Fujii choose game theory for the last row explicitly, noting that RL "often lacks the ability to explicitly explain why a specific decision is considered optimal without supplementary manual analysis."

**Note the irony now that both are held.** Nakahara et al.'s ten agents do *not* model each other — the framework is [[multi-agent-reinforcement-learning|independent MARL]], so teammates enter the state vector but not as reasoning agents. The vault's "multi-agent" RL paper models interaction *less* than its two-agent game-theory paper does. Same group, no mutual citation.

## The Simulator Objection, Revised

**Revised 2026-08-07.** This page previously asserted that the *forward* approach — build an environment, learn inside it — is closed off in football because no environment is faithful enough.

[[google-research-football|GFootball]] is a counter-example to the strong form: a well-engineered football simulator exists, and Nakahara et al. borrow its 19-action vocabulary. What they do **not** borrow is its dynamics, its rewards, or a single frame of simulated experience.

The accurate claim is therefore:

> The forward approach is *available*; what is unavailable is evidence that a policy learned in a simulator transfers to real players.

A fidelity problem, not an availability one. The borrowing pattern is the tell — researchers who trust a simulator's **ontology** but not its **physics** take the action set and leave the rest. Scott, Fujii & Onishi (2022) apparently examine the transfer question directly; cited by Nakahara et al., not held.

## Discounting, Borrowed and Repurposed

$\gamma$ in RL encodes impatience and guarantees convergence of infinite-horizon sums. [[temporal-discounting|Shelopugin]] uses the identical formula for a different purpose: **attribution decay**, on the reasoning that an action not followed by a shot probably did not advance the attack.

Same mathematics, different justification. Conflating them leads to reading a football discount factor as a claim about preferring earlier goals, which it is not.

**A third position now exists.** Nakahara et al. set $\gamma = 1$ — no discounting — safe because reward arrives only at the terminal frame of a bounded episode, but it means credit is spread **flat** across possessions of up to thirty seconds. Three football frameworks, one symbol, values of 0.95, 1, and "not applicable", with no discussion between them. See [[free-parameters-load-bearing]].

## Where the Advantage Function Reappears

[[policy-modelling|Fernández et al.'s]] realised-versus-available gap — policy-weighted EPV 0.032 against best-available 0.112 — is $Q(s,a^*) - V(s)$, computed not to train anything but to tell a coach what was left on the pitch.

Yeung & Fujii reach a convergent conclusion by an unrelated method: passing worth 0.2456 against shooting's 0.0866 when the defender blocks. **Two frameworks, one surface-based and one game-theoretic, both finding observed behaviour diverges systematically from optimal.**

The same algebraic shape appears in [[counterfactual-baseline|counterfactual baselines]], where the reference is a population average or a predicted behaviour rather than an optimum. Three references, three questions, one form.

## RL Proper in This Vault

- **[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. (2023)]]** — **held since 2026-08-07.** Ten independent SARSA agents on J-League tracking; on- and off-ball Q-values at every timestep. The only football source here that optimises a Bellman residual.
- **[[rlhf|RLHF]]** ([[training-lm-follow-instructions-with-human-feedback|InstructGPT]]) — a reward model learned from human preferences, then optimised with PPO. Real policy optimisation, with a [[kl-divergence|KL]] penalty anchoring to the base model. Note even this is partly inverted: the *reward* is learned from behaviour, which is closer to inverse RL.
- **MDP-based decision optimisation** — Rahimian et al. (2022, 2023) and Van Roy et al. (2021) apply MDPs to maximise expected possession outcome. Cited by Yeung & Fujii, not held.
- **Deep RL for team-level valuation** — Liu & Schulte (2018), Liu et al. (2020), Routley & Schulte (2015). The **team-as-one-agent** tradition Nakahara et al. depart from. Cited, not held. See [[multi-agent-reinforcement-learning]].
- **Inverse RL for sports** — Luo, Schulte & Poupart (2020); Rahimian & Toka (2022). Reward functions estimated from behaviour. Cited, not held.

## See Also

- [[markov-game]] · [[game-theory]] · [[value-iteration]] · [[temporal-difference-learning]] · [[temporal-discounting]] · [[policy-modelling]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[google-research-football]]
- [[imitation-learning]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[action-valuation]] · [[expected-possession-value]]
- [[rlhf]] · [[rare-event-proxy-targets]] · [[possession-risk]] · [[xsot]] · [[off-ball-value]]
- [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[training-lm-follow-instructions-with-human-feedback|InstructGPT Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
