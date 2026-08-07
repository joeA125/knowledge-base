---
title: "Temporal-Difference Learning"
type: concept
tags: [temporal-difference, reinforcement-learning, dynamic-programming, markov-model, discounting, deep-learning, rnn, machine-learning, action-valuation, sports-analytics]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 30%
  generated: 5%
  imported: 20%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Temporal-Difference Learning

Learning a value function by updating each estimate toward **the next estimate plus the reward observed in between**, rather than toward a completed return. The defining move is *bootstrapping*: a guess is corrected using another guess.

For state-action values, the target comes from the Bellman equation

$$Q^*(s_t, a_t) = \mathbb{E}_{s_{t+1}, a_{t+1}}\left[r_{t+1} + Q(s_{t+1}, a_{t+1}) \mid s_t, a_t\right]$$

and the loss is the squared residual of that equality:

$$\mathcal{L}_{TD} = \sum_{t \in T} \left(r_{t+1} + Q(s_{t+1}, a_{t+1}) - Q(s_t, a_t)\right)^2$$

## Where It Sits Against What the Vault Already Holds

| Method | Needs a model of dynamics | Needs a complete episode | Bootstraps |
|---|---|---|---|
| [[value-iteration\|Dynamic programming]] | **Yes** | No | Yes |
| Monte Carlo return | No | **Yes** | No |
| **Temporal difference** | **No** | **No** | **Yes** |

TD is what makes value learning possible when transition probabilities are unknown *and* you would rather not wait for the episode to finish. That combination is why it, rather than [[value-iteration|DP]], is what [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] use on football tracking data: the dynamics of 22 interacting players are not writable down, and possessions run to 300 frames.

Contrast [[expected-threat|xT]], which *does* use dynamic programming — because it first coarsens the pitch to a zone grid, at which point the transition matrix is small enough to estimate directly. **The choice between DP and TD in this literature tracks the coarseness of the state representation**, not any deeper commitment.

## SARSA and the On-Policy Choice

SARSA — state, action, reward, state, action — is **on-policy**: the bootstrap target uses the action the agent *actually took next*, $Q(s_{t+1}, a_{t+1})$, not the best available action $\max_a Q(s_{t+1}, a)$.

| | SARSA (on-policy) | Q-learning (off-policy) |
|---|---|---|
| Target uses | The action actually taken | The **best** action |
| Converges toward | Value of the behaviour policy | Value of the **optimal** policy |
| In a valuation context | "What was this worth, given how they play?" | "What would this be worth, played perfectly?" |

That choice is not incidental. **On-policy TD makes the learned $Q$ a description of observed football, not a prescription for better football** — which places this framework alongside [[policy-modelling]] and [[expected-value-possession-framework|Fernández et al.'s]] average-policy stance rather than alongside the optimal-play frameworks. See [[reinforcement-learning]].

It also sits slightly awkwardly with the framework's counterfactual ambitions. The Q-values of unchosen actions are constrained by [[action-supervision]] and network smoothness, not by SARSA updates — since SARSA never targets them.

## Function Approximation and Its Cost

With a tabular $Q$ the residual above is a well-behaved update rule. With a neural network it becomes a loss with a **moving target** — the same parameters produce both $Q(s_t, a_t)$ and $Q(s_{t+1}, a_{t+1})$, so the thing being regressed toward shifts with every step.

Standard mitigations (target networks, replay buffers) are absent in the source, which trains a single [[gated-recurrent-unit|GRU]] on whole possession sequences with $L_1$ [[regularization|regularisation]] and treats the recurrence as its memory. That is defensible at this data scale — 1,669 training sequences — but it means the reported TD losses (0.0034 against 0.0063) measure **self-consistency of a moving estimate**, not accuracy against anything.

The authors are careful about this: they state that the losses permit quantitative comparison of *optimisation* while the model's merit as a description of football can only be assessed qualitatively. That is the right caveat, and it is stronger than most papers in this vault manage.

## The Discount Factor, Set Aside

TD is normally paired with $\gamma < 1$, both for convergence over infinite horizons and to encode impatience. The source sets $\gamma = 1$.

This is safe here — reward arrives **only at the terminal frame**, episodes are capped at 300 frames, so the return is a finite single term. But it has a consequence worth naming: **credit is spread flat across a possession of up to thirty seconds.** An action in second 1 and an action in second 29 of the same possession are, other things equal, credited identically.

That is the opposite pole from [[temporal-discounting|Shelopugin's $\gamma = 0.95$ per second]], which after thirty seconds retains 21% of the weight. Two frameworks valuing football actions, both borrowing the same symbol, at $\gamma = 1$ and $\gamma = 0.95$ with no discussion between them. See [[free-parameters-load-bearing]].

## Why the Vocabulary Recurs Without the Method

Almost every [[action-valuation]] framework in this vault computes $Q(S_i) - Q(S_{i-1})$, which is *shaped* like a TD residual with no reward term. The resemblance is real but shallow: those frameworks fit $Q$ by supervised learning against an eventual outcome label, then difference it. **Differencing a supervised model is not bootstrapping**, because nothing in training ever used one estimate as another's target.

Keeping this straight is the reason [[reinforcement-learning]] exists as a page. Nakahara et al. are the vault's first source where the TD residual is the actual training objective.

## See Also

- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[value-iteration]] · [[markov-game]]
- [[temporal-discounting]] · [[policy-modelling]] · [[action-valuation]] · [[expected-threat]] · [[expected-possession-value]]
- [[gated-recurrent-unit]] · [[regularization]] · [[adam-optimizer]] · [[free-parameters-load-bearing]]
- [[action-valuation-multi-agent-reinforcement-learning|Source Summary]]
