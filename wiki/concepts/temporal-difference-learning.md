---
title: "Temporal-Difference Learning"
type: concept
tags: [temporal-difference, reinforcement-learning, dynamic-programming, markov-model, discounting, deep-learning, rnn, machine-learning, action-valuation, sports-analytics, experience-replay, multi-agent]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.8
provenance:
  extracted: 48%
  inferred: 28%
  generated: 5%
  imported: 19%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Temporal-Difference Learning

Learning a value function by updating each estimate toward **the next estimate plus the reward observed in between**, rather than toward a completed return. The defining move is *bootstrapping*: a guess is corrected using another guess.

For state-action values, the target comes from the Bellman equation

$$Q^*(s_t, a_t) = \mathbb{E}\left[r_{t+1} + \gamma\, Q(s_{t+1}, a_{t+1}) \mid s_t, a_t\right]$$

and the loss is the squared residual of that equality.

## Where It Sits Against What the Vault Holds

| Method | Needs a model of dynamics | Needs a complete episode | Bootstraps |
|---|---|---|---|
| [[value-iteration\|Dynamic programming]] | **Yes** | No | Yes |
| Monte Carlo return | No | **Yes** | No |
| **Temporal difference** | **No** | **No** | **Yes** |

TD is what makes value learning possible when transition probabilities are unknown *and* you would rather not wait for the episode to finish. That combination is why both held RL sources use it on football data: the dynamics of 22 interacting players are not writable down.

Contrast [[expected-threat|xT]], which *does* use dynamic programming — because it first coarsens the pitch to a zone grid, at which point the transition matrix is estimable directly. **The choice between DP and TD in this literature tracks the coarseness of the state representation**, not any deeper commitment.

## Two TD Variants, One Group, Opposite Choices

| | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] |
|---|---|---|
| Update | **SARSA** — on-policy | **DDQN** — off-policy |
| Target | $Q(s_{t+1}, a_{t+1})$, the action taken | $Q(s_{t+1}, a^{\max}_{t+1}; \theta')$, the best action |
| Stabilisers | **None** | Target network, replay, double Q |
| Agent acts | **No** | **Yes**, $\varepsilon$-greedy |

### SARSA and the on-policy choice

SARSA — state, action, reward, state, action — uses the action the agent *actually took next*, not the best available.

| | SARSA (on-policy) | Q-learning (off-policy) |
|---|---|---|
| Converges toward | Value of the behaviour policy | Value of the **optimal** policy |
| In a valuation context | "What was this worth, given how they play?" | "What would this be worth, played perfectly?" |

That choice is not incidental. **On-policy TD makes the learned $Q$ a description of observed football, not a prescription for better football** — which places Nakahara et al. alongside [[policy-modelling]] and [[expected-value-possession-framework|Fernández et al.'s]] average-policy stance rather than alongside the optimal-play frameworks.

It also sits awkwardly with their counterfactual ambitions. The Q-values of unchosen actions are constrained by [[action-supervision]] and network smoothness, not by SARSA updates, since SARSA never targets them.

**Fujii et al. take the other branch** — DDQN's max, appropriate because their agent must eventually choose actions in a simulator. The two papers therefore learn structurally different quantities from the same J-League dataset, and neither notes it. See [[deep-q-network]].

## Function Approximation and Its Cost

With a tabular $Q$ the residual is a well-behaved update rule. With a neural network it becomes a loss with a **moving target** — the same parameters produce both $Q(s_t,a_t)$ and $Q(s_{t+1},\cdot)$, so the regression target shifts with every step.

The standard mitigations are target networks and replay buffers, and [[deep-q-network]] covers them. **Nakahara et al. use neither**, training a single [[gated-recurrent-unit|GRU]] on whole possession sequences with $L_1$ [[regularization|regularisation]] and treating the recurrence as its memory.

> ### `stabilisers-track-the-feedback-loop`
> **The DQN stabiliser set is required in proportion to how much a learning agent influences its own training distribution. Purely offline value estimation from logged behaviour needs little of it.**
> ^[generated: declared on [[deep-q-network]], drawn from the contrast between these two same-group papers. rests-on: source:fujii-ddqn-full-stack, source:nakahara-sarsa-no-stabilisers]

Defensible at 1,669 training sequences with a fixed data distribution — but it means the reported TD losses (0.0034 against 0.0063) measure **self-consistency of a moving estimate**, not accuracy against anything. Nakahara et al. are careful about this, stating the losses permit quantitative comparison of *optimisation* while the model's merit as a description of football can only be assessed qualitatively. That caveat is stronger than most papers in this vault manage.

## The Discount Factor

TD is normally paired with $\gamma < 1$, both for convergence over infinite horizons and to encode impatience.

Nakahara et al. set $\gamma = 1$ — safe, since reward arrives **only at the terminal frame** of an episode capped at 300 frames, so the return is a finite single term. But **credit is spread flat across a possession of up to thirty seconds.**

Fujii et al. define $\gamma \in (0,1]$ and **never report the value used** — less specified than the applied paper that cites them. See [[free-parameters-load-bearing]].

Two frameworks, one symbol, values of 1 and unreported, against [[temporal-discounting|Shelopugin's $\gamma = 0.95$ per second]] which retains 21% of weight after thirty seconds. No discussion between any of them.

**In the $\gamma = 1$ case the bootstrap does the attribution work instead.** The value of an early action is not the discounted terminal reward directly — it is the value of the *next* state, itself bootstrapped forward. Attribution decay becomes implicit in how far the network propagates signal, which nobody measures. See [[gated-recurrent-unit]].

## Why the Vocabulary Recurs Without the Method

Almost every [[action-valuation]] framework in this vault computes $Q(S_i) - Q(S_{i-1})$, which is *shaped* like a TD residual with no reward term. The resemblance is shallow: those frameworks fit $Q$ by supervised learning against an eventual outcome label, then difference it.

**Differencing a supervised model is not bootstrapping**, because nothing in training ever used one estimate as another's target. Keeping this straight is why [[reinforcement-learning]] exists as a page, and why only two held sources qualify as RL proper.

## See Also

- [[reinforcement-learning]] · [[deep-q-network]] · [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[value-iteration]] · [[markov-game]]
- [[temporal-discounting]] · [[policy-modelling]] · [[action-valuation]] · [[expected-threat]] · [[expected-possession-value]] · [[action-space-design]]
- [[gated-recurrent-unit]] · [[regularization]] · [[adam-optimizer]] · [[free-parameters-load-bearing]] · [[domain-adaptation]] · [[imitation-reward-tradeoff]]
- [[hiroshi-nakahara]] · [[keisuke-fujii]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
