---
title: "Reinforcement Learning"
type: concept
tags: [reinforcement-learning, machine-learning, markov-model, dynamic-programming, discounting, policy-modelling, imitation-learning, action-valuation]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/expected_value_possession_framework.md, raw/papers/training-lm-follow-instructions-with-human-feedback.md]
confidence: 0.85
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: draft
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

But almost none of this work is reinforcement learning, because **nobody is learning a policy.** The distinction:

| | Reinforcement learning | Sports valuation |
|---|---|---|
| Goal | Find a better policy | **Measure** the observed one |
| Policy | The output | Fixed, or itself estimated |
| Environment | Interacted with | Only observed |
| Reward | Designed to be optimised | A label to be predicted |

The reasons are practical and worth stating, because they recur in any domain where RL is tempting but unavailable:

- **No simulator.** There is no environment faithful enough to interact with, so the *forward* approach — build a model, learn inside it — is closed off. See [[imitation-learning]].
- **Sparse reward.** Goals are ~0.2% of events, which is too little signal to optimise against. See [[rare-event-proxy-targets]].
- **The counterfactual would be unfounded.** An optimal-policy value function answers "what is this worth to a perfect player?", and no data contains perfect play. See [[policy-modelling]].

So the field takes RL's *analytic* apparatus (value functions, advantage, discounting) and discards its *control* objective. [[expected-value-possession-framework|Fernández, Bornn & Cervone]] state this explicitly: the aim is value under the **average** policy learned from history, not the optimal one.

## Where the Advantage Function Reappears

The most interesting borrowing is [[policy-modelling|Fernández et al.'s]] realised-versus-available gap: policy-weighted EPV 0.032 against best-available 0.112. That difference is $Q(s,a^*) - V(s)$ — an advantage computed not to train anything but to *tell a coach what was left on the pitch*.

The same shape appears in [[counterfactual-baseline|counterfactual baselines]], where the reference is a population average or a predicted behaviour rather than an optimum. Three different references, three different questions, one algebraic form.

## Discounting, Borrowed and Repurposed

$\gamma$ in RL encodes impatience and guarantees convergence of infinite-horizon sums. [[temporal-discounting|Shelopugin]] uses the identical formula for a different purpose: **attribution decay**, on the reasoning that an action not followed by a shot probably did not advance the attack.

Same mathematics, different justification. Conflating the two leads to reading a football discount factor as a claim about preferring earlier goals, which it is not.

## RL Proper in This Vault

Two genuine instances:

- **[[rlhf|RLHF]]** ([[training-lm-follow-instructions-with-human-feedback|InstructGPT]]) — a reward model learned from human preferences, then optimised with PPO. Real policy optimisation, with a [[kl-divergence|KL]] penalty anchoring to the base model.
- **Multi-agent RL for on/off-ball valuation** (arXiv:2305.17886) — cited, not held.

Note that even RLHF is partly inverted: the *reward* is learned from behaviour, which is closer to inverse RL than to classical RL.

## See Also

- [[markov-game]] · [[value-iteration]] · [[temporal-discounting]] · [[policy-modelling]]
- [[imitation-learning]] · [[counterfactual-baseline]] · [[action-valuation]] · [[expected-possession-value]]
- [[rlhf]] · [[rare-event-proxy-targets]] · [[possession-risk]]
- [[training-lm-follow-instructions-with-human-feedback|InstructGPT Summary]]
