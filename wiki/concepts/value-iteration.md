---
title: "Value Iteration"
type: concept
tags: [dynamic-programming, reinforcement-learning, markov-model, machine-learning, approximation]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Value Iteration

Value iteration is a dynamic-programming algorithm for computing the value function of a Markov process. Starting from an arbitrary initialisation, it repeatedly applies a Bellman backup until values converge:

$$V_{k+1}(s) \leftarrow R(s) + \gamma \sum_{s'} T(s \to s') \, V_k(s')$$

Each sweep propagates value one step further backward through the state space, so after $k$ iterations $V_k(s)$ reflects rewards reachable within $k$ steps.

## Convergence

Under a discount factor $\gamma < 1$ (or with absorbing states reached with probability 1), the Bellman operator is a contraction mapping, so iteration converges to a unique fixed point regardless of initialisation. Convergence is geometric, at rate $\gamma$.

## Use in Expected Threat

[[expected-threat|xT]] is computed by value iteration over pitch zones:

$$xT(z) = s_z \cdot xG(z) + m_z \cdot \sum_{z'} T_{z \to z'} \cdot xT(z')$$

The correspondence to the general form is direct:
- **States** = pitch zones
- **Reward** = $s_z \cdot xG(z)$, the immediate expected goal payoff from shooting here
- **Transitions** = $m_z \cdot T_{z \to z'}$, moving the ball elsewhere
- **Absorption** = a shot or a turnover ends the possession, so no discount factor is needed

Initialising all $xT(z) = 0$ gives a clean interpretation: **after iteration $i$, $xT(z)$ is the probability of scoring within the next $i$ actions.** Each iteration literally looks one action further ahead. [[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] report convergence after 6 iterations on a $16 \times 12$ grid — meaning goal-scoring threat in soccer effectively propagates about six actions.

## Relation to Policy Iteration

Value iteration and policy iteration are the two classical dynamic-programming methods for [[markov-model|MDPs]]. Policy iteration alternates full policy evaluation with policy improvement; value iteration collapses these into a single backup per sweep. Value iteration typically needs more sweeps but each is far cheaper.

Note that xT performs *evaluation*, not *optimisation* — the transition probabilities are estimated from observed play, so xT computes the value of how teams actually behave rather than of optimal play. There is no $\max$ over actions in the recursion.

## Contrast with Supervised Estimation

The alternative to solving a recursion is to learn state values directly from labelled outcomes, as [[vaep]] does with [[gradient-boosting]] classifiers. The trade-off:

| | Value iteration (xT) | Supervised (VAEP) |
|---|---|---|
| Requires | Transition matrix over a discrete state space | Labelled examples with features |
| State space | Must be small enough to enumerate | Arbitrary, continuous features fine |
| Output | Exact fixed point | Approximation with generalisation error |
| [[interpretability]] | High — values are explicit per state | Low — implicit in the model |

The enumeration requirement is why xT must discretise the pitch into zones, which is in turn the root of most of its limitations.

## Broader Context

Value iteration underpins classical [[reinforcement-learning]] and, in approximate form, deep RL. The same backup structure appears in the reward modelling of [[rlhf]], and in the transition-matrix algebra used to compute $\mathbb{E}[X \mid C_{\delta_t}]$ in the [[expected-possession-value|EPV]] model's coarsened chain.

## See Also

- [[expected-threat]]
- [[markov-game]]
- [[reinforcement-learning]]
- [[expected-possession-value]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]]
