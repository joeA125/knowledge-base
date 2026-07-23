---
title: "VAEP (Valuing Actions by Estimating Probabilities)"
type: concept
tags: [machine-learning, sports-analytics, statistics, evaluation, player-evaluation, probabilistic-classification, markov-model]
sources: [raw/papers/evaluating-football-player-actions.md]
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

VAEP ([[evaluating-football-player-actions|Decroos et al., 2019]]) is a framework for assigning a value to every on-ball action in a soccer game based on its impact on the probability of scoring and conceding in the near future.

## Core Idea

For each game state $S_i$ (the sequence of actions so far), estimate the probability of the team in possession scoring ($P_{scores}$) and conceding ($P_{concedes}$) within the next $k=10$ actions. An action's value is the change it causes in these probabilities:

$$V(a_i) = \underbrace{\Delta P_{scores}(a_i)}_{\text{offensive value}} + \underbrace{(-\Delta P_{concedes}(a_i))}_{\text{defensive value}}$$

A positive value means the action helped the team (increased scoring chance or decreased conceding chance); a negative value means it hurt them.

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

Two [[probabilistic-classification|probabilistic classifiers]] ([[gradient-boosting|CatBoost]]) are trained to estimate $P_{scores}$ and $P_{concedes}$, using features from the previous 3 actions (SPADL features + complex features + game context). CatBoost outperforms Logistic Regression, Random Forest, and XGBoost on both [[probability-calibration|Brier score and ROC AUC]], attributed to its intelligent handling of categorical features.

Because action values are computed by **summing and subtracting** these probabilities, [[probability-calibration|calibration]] is essential — miscalibrated probabilities would propagate directly into the action values. This is why the paper evaluates with the Brier score (a proper scoring rule), not just discrimination metrics.

## Modelling as a Markov Game

VAEP fits within a line of related work that models sports matches as a [[markov-game]] — the framework for sequential, multi-agent, adversarial decision-making (Routley & Schulte for ice hockey, [[multiresolution-stochastic-process-nba-possessions|Cervone et al.]] for basketball). The core assumption inherited from that framing is the Markov property: near-future scoring/conceding probability depends on the current game state, not the full history. VAEP approximates the state by the last 3 actions — a truncated representation that trades some history for fixed-length features — and estimates state values directly with supervised learning rather than solving the game for equilibrium policies. Unlike zone-based Markov models, VAEP uses exact action locations.

## Relation to Basketball's EPV

[[expected-possession-value|EPV]] answers the same question for basketball, but from a different data regime and with stronger theoretical guarantees:

| | VAEP | [[expected-possession-value\|EPV]] |
|---|---|---|
| Data | [[event-stream-data]] | [[optical-tracking-data]], 25 Hz |
| Model | [[gradient-boosting]] classifiers | Bayesian [[stochastic-process]] model |
| Output | Per-action value $V(a_i)$ | Continuous curve $\nu_t$ |
| [[martingale]] structure | Not guaranteed | Guaranteed by construction |
| Aggregation | Values sum directly per player | Requires a relative baseline (EPVA) |
| Cost | Modest | Very high (461 processors) |
| Portability | Widely available data | Requires camera installations |

The martingale contrast is the sharpest difference. Because EPV is a genuine conditional expectation, a player's raw EPV changes average to zero, forcing EPVA to compare against a hypothetical league-average player. VAEP's values, produced by differencing two independently trained classifiers, carry no such constraint — so they can be summed directly, at the cost of losing the formal consistency guarantee.

## Player Ratings

Action values aggregate into per-90-minute player ratings:

$$rating(p) = \frac{90}{m} \sum_{a_i \in A_p^T} V(a_i)$$

Ratings can be decomposed by action type to characterise playing style — e.g., comparing Neymar's dribble value (0.16) with Coutinho's pass value (0.28). This supports [[player-evaluation]] use cases: scouting, recruitment, and style matching.

## Relation to Expected Goals

[[expected-goals|Expected Goals (xG)]] is a special case of VAEP: computing the xG of a shot is equivalent to estimating $P_{scores}$ at the game state immediately before the shot. VAEP generalises this to all action types.

## Limitations

- Only values on-ball actions — defensive positioning, pressing, and off-ball movement are invisible. (The [[optical-tracking-data|tracking-based]] EPV model partially addresses this, valuing off-ball players as potential pass targets.)
- Cross-league and cross-club comparisons are difficult (it's easier to perform valuable actions in weaker leagues or at stronger clubs).

## See Also

- [[spadl]]
- [[expected-goals]]
- [[expected-possession-value]]
- [[event-stream-data]]
- [[gradient-boosting]]
- [[probability-calibration]]
- [[markov-game]]
- [[martingale]]
- [[evaluating-football-player-actions|Source Summary]]
- [[game-state-reconstruction]]
