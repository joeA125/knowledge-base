---
title: "Reinforcement Learning"
type: concept
tags: [reinforcement-learning, machine-learning, markov-model, dynamic-programming, discounting, policy-modelling, imitation-learning, game-theory, action-valuation]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/expected_value_possession_framework.md, raw/papers/training-lm-follow-instructions-with-human-feedback.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.85
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Reinforcement Learning

Learning to act by interacting with an environment and receiving reward. Formally, finding a policy $\pi(a|s)$ maximising expected cumulative discounted reward in a [[markov-game|Markov decision process]].

This page exists because RL vocabulary is used throughout the vault — value functions, advantage, discounting, policy — by work that is **mostly not doing reinforcement learning.** Being clear about which parts are borrowed and which are not prevents a recurring confusion.

## The Core Objects

| Object | Definition | Role |
|---|---|---|
| State value $V(s)$ | Expected return from $s$ under $\pi$ | "How good is this situation?" |
| Action value $Q(s,a)$ | Expected return from taking $a$ in $s$ | "How good is this move?" |
| **Advantage** $A(s,a) = Q(s,a) - V(s)$ | How much better than average | "Was that a good decision?" |
| Discount factor $\gamma$ | Geometric weight on future reward | Convergence and impatience |

Solved by [[value-iteration|dynamic programming]] when the model is known, or by sampling (Q-learning, policy gradients, actor-critic) when it is not.

## What Sports Analytics Borrows — and What It Does Not

Every [[action-valuation]] framework in this vault uses the equation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

which is structurally an **advantage**. And [[expected-possession-value|EPV]] is structurally a state-value function. The vocabulary is exact.

But almost none of this work is reinforcement learning, because **nobody is learning a policy by interaction.**

| | Reinforcement learning | Sports valuation |
|---|---|---|
| Goal | Find a better policy | **Measure** the observed one |
| Policy | The output | Fixed, or itself estimated |
| Environment | Interacted with | Only observed |
| Reward | Designed to be optimised | A label to be predicted |

The reasons are practical and recur in any domain where RL is tempting but unavailable:

- **No simulator.** There is no environment faithful enough to interact with, so the *forward* approach — build a model, learn inside it — is closed off. See [[imitation-learning]].
- **Sparse reward.** Goals are ~0.2% of events, too little signal to optimise against. See [[rare-event-proxy-targets]].
- **Extrapolation risk in the counterfactual.** An optimal-policy value function must evaluate actions nobody took. Over a large action space that is extrapolation, not estimation.

So the field takes RL's *analytic* apparatus (value functions, advantage, discounting) and mostly discards its *control* objective. [[expected-value-possession-framework|Fernández, Bornn & Cervone]] state this explicitly: the aim is value under the **average** policy learned from history.

## Where Optimal Policy *Is* Recovered

**Correction, 2026-07-27.** An earlier revision of this page stated flatly that this literature does not solve for optimal policy. That is too strong.

[[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] do — not with RL, but with [[game-theory]]. The route around the third objection above is to **shrink the strategy space until every profile is enumerable**: the shooter chooses {Shoot, Pass}, the defender {Block, Not Block}, and payoffs for the two unobserved profiles are estimated by re-running the models with the closest defender removed.

That converts extrapolation into estimation, and yields a Nash equilibrium of (Pass, Block).

The general lesson is worth separating from the football: **the barrier to optimal-policy analysis is the size of the action space, not the observational nature of the data.** Where the space can be coarsened to a handful of options, optimal analysis is available; the cost is that the answer concerns the coarsened game. "Pass" collapsing ten possible recipients into one option is exactly that cost.

## Game Theory as the Alternative Route

The two are often conflated because both concern optimal action:

| | Reinforcement learning | [[game-theory\|Game theory]] |
|---|---|---|
| Other agents | Folded into the environment | **Modelled as agents** |
| Solution concept | Optimal policy against a fixed world | **Equilibrium** against a reasoning opponent |
| Needs | Reward signal, many interactions | Payoffs for every strategy profile |
| Explains *why* | Poorly, without further analysis | By construction — the payoff table is the explanation |

Yeung & Fujii choose game theory for the last row explicitly, noting that RL "often lacks the ability to explicitly explain why a specific decision is considered optimal without supplementary manual analysis."

## Where the Advantage Function Reappears

[[policy-modelling|Fernández et al.'s]] realised-versus-available gap — policy-weighted EPV 0.032 against best-available 0.112 — is $Q(s,a^*) - V(s)$, computed not to train anything but to tell a coach what was left on the pitch.

Yeung & Fujii reach a convergent conclusion by an unrelated method: passing worth 0.2456 against shooting's 0.0866 when the defender blocks. **Two frameworks, one surface-based and one game-theoretic, both finding observed behaviour diverges systematically from optimal.**

The same algebraic shape appears in [[counterfactual-baseline|counterfactual baselines]], where the reference is a population average or a predicted behaviour rather than an optimum. Three references, three questions, one form.

## Discounting, Borrowed and Repurposed

$\gamma$ in RL encodes impatience and guarantees convergence of infinite-horizon sums. [[temporal-discounting|Shelopugin]] uses the identical formula for a different purpose: **attribution decay**, on the reasoning that an action not followed by a shot probably did not advance the attack.

Same mathematics, different justification. Conflating them leads to reading a football discount factor as a claim about preferring earlier goals, which it is not.

## RL Proper in This Vault

- **[[rlhf|RLHF]]** ([[training-lm-follow-instructions-with-human-feedback|InstructGPT]]) — a reward model learned from human preferences, then optimised with PPO. Real policy optimisation, with a [[kl-divergence|KL]] penalty anchoring to the base model. Note even this is partly inverted: the *reward* is learned from behaviour, which is closer to inverse RL.
- **MDP-based decision optimisation** — Rahimian et al. (2022, 2023) and Van Roy et al. (2021) apply MDPs to maximise expected possession outcome. Cited by Yeung & Fujii, not held.
- **Multi-agent RL for on/off-ball valuation** (arXiv:2305.17886) — cited, not held.

## See Also

- [[markov-game]] · [[game-theory]] · [[value-iteration]] · [[temporal-discounting]] · [[policy-modelling]]
- [[imitation-learning]] · [[counterfactual-baseline]] · [[action-valuation]] · [[expected-possession-value]]
- [[rlhf]] · [[rare-event-proxy-targets]] · [[possession-risk]] · [[xsot]]
- [[training-lm-follow-instructions-with-human-feedback|InstructGPT Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
