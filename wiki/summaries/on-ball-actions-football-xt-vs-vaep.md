---
title: "Valuing On-the-Ball Actions in Soccer: A Critical Comparison of xT and VAEP — Source Summary"
type: source_summary
tags: [sports-analytics, action-valuation, player-evaluation, event-stream-data, evaluation, reliability, interpretability, markov-model, machine-learning]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Valuing On-the-Ball Actions in Soccer: A Critical Comparison of xT and VAEP

**Authors:** [[maaike-van-roy]], [[pieter-robberechts]], [[tom-decroos]], [[jesse-davis]]
**Affiliation:** KU Leuven, Department of Computer Science
**Published:** 2020 (AAAI)

## Key Contribution

A direct conceptual, qualitative, and quantitative comparison of two leading [[action-valuation]] frameworks: [[expected-threat]] (xT) and [[vaep]]. Notable as one of the first works to compare such models against each other rather than only against traditional statistics. Written by the VAEP authors themselves — including a finding unfavourable to their own model.

## Motivating Observation

Most soccer metrics measure only shots and assists, which together represent **less than 1% of all on-the-ball actions**. Action valuation frameworks exist to value the other 99%.

## Three Styles of Action Valuation

The paper offers a useful taxonomy of event-stream approaches:

| Style | Mechanism | Examples |
|---|---|---|
| **Count-based** | Weighted sum of action-type counts; weights learned by correlating with match outcome | McHale & Scarf; PlayerRank |
| **Possession-based (EPV)** | Divide match into possessions; value ball-progressing actions via a Markov model | Rudd; Mackay; [[expected-threat\|xT]]; Yam |
| **Action-based** | Value a broader action set with full action and game context, framed as binary classification | [[vaep]] |

Both of the latter two share the same core equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

They differ entirely in how they represent and value the game state $Q$.

## The Two Models

### xT
State $S_i$ = the pitch zone containing the ball. Solved by [[value-iteration]] on:

$$xT(z) = s_z \cdot xG(z) + m_z \cdot \sum_{z'} T_{z \to z'} \cdot xT(z')$$

Values a successful action as $xT(z') - xT(z)$. Experiments use a $16 \times 12$ grid, converging in 6 iterations.

### VAEP
State $S_i$ = the last three actions, described by action features, context features, and game-state features. Values an action as $\Delta P_{score} - \Delta P_{concede}$, with each probability from an XGBoost model.

## Two Axes of Difference

### Location-based vs feature-based
- **xT can only value ball-progressing actions** — passes, dribbles, crosses. Tackles, interceptions, and take-ons within a zone are invisible to it. Hence the label "ball-progression models".
- **VAEP captures action context**, e.g. that a shot is easier when set up by a through ball than after dribbling past defenders.
- **VAEP captures game context** — score difference, time remaining.
- **xT is interpretable**: $M \cdot N$ values, one per zone. VAEP requires a function approximator, so explaining a specific valuation is not straightforward. ([[interpretability]] trade-off.)

### Possession-based vs window-based
- **VAEP captures risk**: because it looks beyond turnovers, it can value how an action changes the chance of *conceding*. xT models offence only, so it ignores the counterattack risk of, say, a square pass in midfield.
- **VAEP can value failed actions accurately**: kicking the ball out of bounds to escape a press is better than a loose forward kick, but both are simply possession losses to xT.

## Qualitative Comparison — Four Action Types

| Action | Finding |
|---|---|
| Backward pass into own box (~19/game) | xT captures neither risk nor reward; all own-half zones ≈ 0. VAEP assigns diverse positive and negative values. |
| First ball progression of a counterattack (~2–3/game) | Only VAEP captures it; xT values are extremely skewed to zero. |
| Forward dribble inside the box (~4/game) | xT assigns zero to many, since short dribbles do not cross a zone boundary — yet near goal, small displacements matter. |
| Forward pass to the box border (~5/game) | xT values these *higher* than VAEP. The authors decline to say which is right: ground truth is unavailable. |

## Player Rankings (2018/19 EPL, StatsBomb, [[spadl]] format)

Both rank Eden Hazard first. Beyond that they diverge sharply:

- **Sergio Agüero**: 19th by VAEP, 109th by xT. He overperforms on shots — a skill VAEP rewards heavily and xT ignores entirely, since xT gives no credit for shooting.
- **Alexis Sánchez**: 7th by xT, 106th by VAEP. His xG/90 more than halved after moving to Manchester United while his key passes held steady — so he still created threat without finishing it.
- **Virgil van Dijk**: 81st (VAEP) and 142nd (xT). Both frameworks structurally favour attackers.

Jaccard similarity between top-$k$ sets: **0.18** at $k=5$, **0.48** at $k=25$, dipping to 0.35 at $k=50$ before climbing. See [[jaccard-index]].

## Correlation with Traditional Metrics

| | Goals/90 | Assists/90 |
|---|---|---|
| VAEP | $\rho = 0.41$ | $\rho = 0.33$ |
| xT | $\rho = 0.26$ | $\rho = 0.53$ |

VAEP leans toward shooting; xT leans toward playmaking. Both deviate substantially from goals and assists, which is the point — they surface information those metrics miss.

## Robustness — The Striking Result

Splitting the season into two random disjoint halves and correlating each player's rating across them ([[split-half-reliability]]):

| Model | Pearson $\rho$ |
|---|---|
| xT | **0.89** |
| VAEP (all actions) | **0.25** |
| VAEP (offensive value, ball-progressing actions only) | 0.59 |

xT's ratings are dramatically more stable. The authors attribute this to two causes: VAEP assigns high values to goals, so a difference of three goals can double or halve a defender's rating; and players are highly consistent in *where* they perform *which* action types, so a purely zonal metric is naturally stable.

Notably, restricting VAEP to xT's action set and dropping defensive value only recovers part of the gap ($0.25 \to 0.59$, still well below 0.89) — so the difference is not merely a matter of scope.

## Conclusion

Neither model dominates. xT is more interpretable and far more robust; VAEP captures risk, context, and a wider action set. Both give insight beyond goals and assists.

## See Also

- [[expected-threat]]
- [[vaep]]
- [[action-valuation]]
- [[split-half-reliability]]
- [[value-iteration]]
- [[action-valuation-frameworks-compared]]
