---
title: "VAEP (Valuing Actions by Estimating Probabilities)"
type: concept
tags: [machine-learning, sports-analytics, statistics, evaluation, player-evaluation, probabilistic-classification, markov-model, action-valuation, reliability]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-23
---

# VAEP (Valuing Actions by Estimating Probabilities)

VAEP ([[evaluating-football-player-actions|Decroos et al., 2019]]) is a framework for assigning a value to every on-ball action in a soccer game based on its impact on the probability of scoring and conceding in the near future. It is the canonical *action-based* approach in the [[action-valuation]] taxonomy.

## Core Idea

For each game state $S_i$ (the sequence of actions so far), estimate the probability of the team in possession scoring ($P_{scores}$) and conceding ($P_{concedes}$) within the next $k=10$ actions. An action's value is the change it causes in these probabilities:

$$V(a_i) = \underbrace{\Delta P_{scores}(a_i)}_{\text{offensive value}} + \underbrace{(-\Delta P_{concedes}(a_i))}_{\text{defensive value}}$$

A positive value means the action helped the team; a negative value means it hurt them.

## Why VAEP Improves on Traditional Metrics

| Limitation of traditional metrics | How VAEP addresses it |
|---|---|
| Only count goals and assists | Values all 21 [[spadl]] action types (passes, dribbles, tackles, clearances, etc.) |
| Fixed value regardless of context | Context-aware: same action type valued differently based on location, game state, preceding actions |
| Only immediate effects | Considers effects up to 10 actions ahead via probabilistic classification |

This is why VAEP identifies players like De Bruyne and Hazard who are invisible to goals/assists metrics but whose passing and dribbling consistently raise their team's scoring probability.

## Data Foundation

VAEP is built on [[spadl]] representations of [[event-stream-data]]. The fixed 9-attribute action format makes it possible to construct the fixed-length feature vectors that the probability estimators require.

## Probability Estimation

Two [[probabilistic-classification|probabilistic classifiers]] ([[gradient-boosting|CatBoost]]; XGBoost in later work) estimate $P_{scores}$ and $P_{concedes}$ from features of the previous 3 actions — action characteristics, action context, and game context (score difference, time remaining). CatBoost outperforms Logistic Regression, Random Forest, and XGBoost on both [[probability-calibration|Brier score and ROC AUC]].

Because action values are computed by **summing and subtracting** these probabilities, [[probability-calibration|calibration]] is essential — miscalibrated probabilities would propagate directly into the action values.

## Modelling as a Markov Game

VAEP fits within a line of work modelling sports matches as a [[markov-game]] (Routley & Schulte for ice hockey, [[multiresolution-stochastic-process-nba-possessions|Cervone et al.]] for basketball). It adopts the Markov assumption but not the process machinery: the state is approximated by the last 3 actions, and state values are estimated by supervised learning rather than by solving a recursion.

## Comparison with xT

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] — including VAEP's own authors — compare VAEP directly against [[expected-threat|xT]]:

| | VAEP | [[expected-threat\|xT]] |
|---|---|---|
| Game state | Last 3 actions + context | Ball's pitch zone only |
| Actions valued | All 21 types | Ball progression only |
| Models risk (conceding) | Yes | No |
| Values finishing | Yes | No |
| [[interpretability]] | Low (function approximator) | High ($M \cdot N$ values) |
| [[split-half-reliability\|Reliability]] | **ρ = 0.25** | **ρ = 0.89** |
| Correlates with | Goals/90 (ρ=0.41) | Assists/90 (ρ=0.53) |

**Where VAEP wins.** It captures the risk–reward trade-off of backward passes into one's own box; it values counterattack initiation, which xT scores near zero; and it values short dribbles inside the penalty box that do not cross a zone boundary.

**Where VAEP loses.** Its player ratings replicate poorly across random halves of a season ($\rho = 0.25$ against xT's 0.89). Restricting VAEP to xT's action set and offensive dimension only lifts this to 0.59 — so the instability comes from the richer state representation itself, not merely from broader scope. The chief culprit is that goals carry large VAEP values and are rare: three goals can double or halve a defender's rating.

This is a bias–variance trade-off. VAEP's context sensitivity is exactly what makes it useful for analysing specific passages of play and exactly what makes its season-long ratings noisy.

## Relation to Basketball's EPV

[[expected-possession-value|EPV]] answers the same question from [[optical-tracking-data]] with a Bayesian [[stochastic-process]] model. Because EPV is a genuine conditional expectation it is a [[martingale]], so a player's raw EPV changes average to zero, forcing a relative baseline (EPVA). VAEP's values, produced by differencing two independently trained classifiers, carry no such constraint and can be summed directly — at the cost of losing that consistency guarantee.

## Player Ratings

$$rating(p) = \frac{90}{m} \sum_{a_i \in A_p^T} V(a_i)$$

Ratings decompose by action type to characterise playing style — Neymar's dribble value (0.16) against Coutinho's pass value (0.28) — supporting [[player-evaluation]] use cases in scouting and recruitment.

## Relation to Expected Goals

[[expected-goals|xG]] is a special case: computing the xG of a shot is equivalent to estimating $P_{scores}$ immediately before the shot. VAEP generalises this to all action types.

## Limitations

- Only values on-ball actions — pressing, marking, and off-ball movement are invisible.
- Structurally favours attackers; Virgil van Dijk ranks 81st.
- Cross-league and cross-club comparison is difficult.
- Low interpretability: no straightforward explanation for why a given action received a given value.

## See Also

- [[action-valuation]]
- [[expected-threat]]
- [[spadl]]
- [[expected-goals]]
- [[expected-possession-value]]
- [[split-half-reliability]]
- [[event-stream-data]]
- [[gradient-boosting]]
- [[probability-calibration]]
- [[markov-game]]
- [[martingale]]
- [[action-valuation-frameworks-compared]]
- [[evaluating-football-player-actions|VAEP Source Summary]]
- [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Comparison Summary]]
