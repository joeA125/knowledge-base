---
title: "Value Iteration"
type: concept
tags: [dynamic-programming, reinforcement-learning, markov-model, machine-learning, approximation, policy-modelling, game-theory]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Value Iteration

Value iteration is a dynamic-programming algorithm for computing the value function of a Markov process. Starting from an arbitrary initialisation, it repeatedly applies a Bellman backup until values converge:

$$V_{k+1}(s) \leftarrow R(s) + \gamma \sum_{s'} T(s \to s') \, V_k(s')$$

Each sweep propagates value one step further backward through the state space, so after $k$ iterations $V_k(s)$ reflects rewards reachable within $k$ steps.

## Convergence

Under a discount factor $\gamma < 1$ (or with absorbing states reached with probability 1), the Bellman operator is a contraction mapping, so iteration converges to a unique fixed point regardless of initialisation. Convergence is geometric, at rate $\gamma$. See [[temporal-discounting]] for the two quite different reasons a discount factor appears in this literature.

## Use in Expected Threat

[[expected-threat|xT]] is computed by value iteration over pitch zones:

$$xT(z) = s_z \cdot xG(z) + m_z \cdot \sum_{z'} T_{z \to z'} \cdot xT(z')$$

The correspondence to the general form is direct:
- **States** = pitch zones
- **Reward** = $s_z \cdot xG(z)$, the immediate expected goal payoff from shooting here
- **Transitions** = $m_z \cdot T_{z \to z'}$, moving the ball elsewhere
- **Absorption** = a shot or a turnover ends the possession, so no discount factor is needed

Initialising all $xT(z) = 0$ gives a clean interpretation: **after iteration $i$, $xT(z)$ is the probability of scoring within the next $i$ actions.** [[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] report convergence after 6 iterations on a $16 \times 12$ grid — meaning goal-scoring threat in soccer effectively propagates about six actions.

This is the standard estimation route for the possession-based family of [[expected-possession-value]] models in soccer.

## Evaluation, Not Optimisation

Note that xT performs *evaluation*, not *optimisation* — the transition probabilities are estimated from observed play, so it computes the value of how teams actually behave. **There is no $\max$ over actions in the recursion.**

That is the standard position in this literature rather than an oversight; see [[policy-modelling]].

**Correction, 2026-07-27.** An earlier revision of this page implied optimisation was unavailable in football analytics generally. [[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] compute an optimal policy, though by a different route — [[game-theory|game theory]] over a two-action strategy space rather than dynamic programming over pitch zones.

The contrast is instructive about what each method needs:

| | Value iteration (xT) | Game-theoretic solution |
|---|---|---|
| State/strategy space | 192 zones, enumerable | 2 × 2 profiles, enumerable |
| Requires | Transition matrix over states | Payoff for **every** profile |
| Unobserved cases | Handled by the recursion | Must be **estimated counterfactually** |
| Models the opponent | No — folded into transitions | **Yes, as an agent** |
| Output | Value of observed behaviour | **Equilibrium** |

Both need an enumerable space. The difference is that value iteration derives unobserved values by propagation through the transition structure, whereas a game solution needs each payoff supplied independently — which is why the strategy space must be far smaller.

## Relation to Policy Iteration

Value iteration and policy iteration are the two classical dynamic-programming methods for [[markov-game|MDPs]]. Policy iteration alternates full policy evaluation with policy improvement; value iteration collapses these into a single backup per sweep. Value iteration typically needs more sweeps but each is far cheaper.

## Contrast with Supervised Estimation

The alternative to solving a recursion is to learn state values directly from labelled outcomes, as [[vaep]] does with [[gradient-boosting]] classifiers:

| | Value iteration (xT) | Supervised (VAEP) |
|---|---|---|
| Requires | Transition matrix over a discrete state space | Labelled examples with features |
| State space | Must be small enough to enumerate | Arbitrary, continuous features fine |
| Output | Exact fixed point | Approximation with generalisation error |
| [[interpretability]] | High — values are explicit per state | Low — implicit in the model |

The enumeration requirement is why xT must discretise the pitch into zones, which is in turn the root of most of its limitations.

## Broader Context

Value iteration underpins classical [[reinforcement-learning]] and, in approximate form, deep RL. The same backup structure appears in the reward modelling of [[rlhf]], and in the transition-matrix algebra used to compute $\mathbb{E}[X \mid C_{\delta_t}]$ in [[martingale-epv|the basketball EPV model's]] coarsened chain.

## See Also

- [[expected-threat]] · [[expected-possession-value]] · [[markov-game]] · [[absorbing-markov-chain]]
- [[reinforcement-learning]] · [[game-theory]] · [[policy-modelling]] · [[temporal-discounting]] · [[martingale-epv]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
