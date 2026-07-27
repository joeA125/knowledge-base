---
title: "Action Valuation"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, markov-model, evaluation, event-stream-data, time-series, recruitment]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Action Valuation

Action valuation is the task of assigning a numeric value to each individual action a player performs, reflecting how much that action improved or damaged their team's prospects. It exists because traditional statistics measure almost nothing: shots and assists together account for **less than 1% of all on-the-ball actions** in soccer.

## The Unifying Equation

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] observe that essentially all modern approaches share one form. Treating a match as a sequence of actions $a_1, \dots, a_n$, where action $a_i$ moves the game from state $S_{i-1}$ to $S_i$:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

An action is worth the change in game-state quality it produces. Every framework in this family differs only in **how it represents $S$** and **how it computes $Q$**.

This is deliberately reminiscent of value functions in [[reinforcement-learning]]: $Q$ plays the role of a state-value function and $V(a_i)$ the role of an advantage.

## Three Styles

### Count-based
Assign a weight to each action type, then take a weighted sum of how many times a player performed each type. Weights are learned by correlating counts with match outcomes or goals.

*Examples:* McHale & Scarf (2007); PlayerRank (Pappalardo et al., 2019).

*Limitation:* aggregates actions with no regard for context. A pass is a pass, wherever and whenever it happens.

### Possession-based
Split the match into possessions, then value ball-progressing actions by how they change the chance of a goal *within that possession*. Almost all such models are [[markov-model|Markov models]] over a discretised state space. This is the family the soccer literature labels "[[expected-possession-value|expected possession value]] approaches".

*Examples:* Rudd (2011); Mackay (2017); [[expected-threat|xT]] (Singh, 2019); Yam (2019).

*Limitation:* stops at the turnover, so cannot model risk; only values actions that progress the ball.

### Action-based
Value a broader set of actions using rich features of both the action and the game context, framed as supervised binary classification.

*Example:* [[vaep]] (Decroos et al., 2019).

*Limitation:* loses [[interpretability]]; empirically less stable ([[split-half-reliability]]).

## What Distinguishes the Approaches

The design space has a few recurring axes:

| Axis | Options | Consequence |
|---|---|---|
| State representation | Zone only → last-$k$ actions → full continuous tracking | Determines which actions can be valued at all |
| Horizon | Current possession → next $k$ actions → end of possession | Determines whether risk (conceding) is modelled |
| Estimation | Dynamic programming → supervised learning → Bayesian process model | Determines interpretability and cost |
| Data | [[event-stream-data]] → [[optical-tracking-data]] | Determines availability and off-ball coverage |
| **Outcome visibility** | Outcome features included → withheld | Determines whether execution or *decision* is valued |
| **Credit assignment** | Fixed action window → time-decayed | Determines how value is attributed backwards from a goal |

The last two axes come from [[football-performance-time-series|Mendes-Neves et al.]] and are underexplored relative to the first four.

**Outcome visibility** is the more consequential. Most frameworks silently conflate *was that a good decision* with *was that well executed*. Partitioning features on whether they encode post-commitment information separates the two — see [[intent-vs-outcome-valuation]]. It is nearly free to implement: the same model, the same data, one ablated feature group.

**Credit assignment** concerns how value propagates back from scoring events. VAEP's fixed $k=10$ action window treats the tenth-previous action as fully in scope and the eleventh as fully out; a time-decayed label makes that boundary continuous.

## The Aggregation Step

Almost every framework ends by summing action values into a per-90 rating. This step is rarely examined and is where [[player-rating-time-series|a season of variation gets discarded]] — form, development, and style change all collapse into one number.

It also interacts with reliability. Since [[split-half-reliability|VAEP's instability]] is measured *on the aggregate*, some of what looks like metric noise may be real within-season variation in the player. No vault source separates the two.

## Cross-Sport Lineage

The problem is not soccer-specific, and soccer arrived late because its low scoring rate and sparse on-ball actions make it especially hard. Antecedents:

- American football — Romer (2006)
- Basketball — [[martingale-epv|Cervone et al.]] (2014, 2016)
- Ice hockey — Routley & Schulte (2015); Liu & Schulte (2018)
- Rugby — Kempton et al. (2016)

## Common Structural Bias

Every framework in this family rewards offensive actions more richly than defensive ones, because value is defined via proximity to scoring. Virgil van Dijk — an elite centre-back — ranks 81st by VAEP and 142nd by xT. This is a limitation of the paradigm, not of any one model.

A partly separate cause, noted by Mendes-Neves et al.: [[event-stream-data|event data]] simply lacks the context to judge tackles and interceptions well. Some of the defensive undervaluation is a **data** limitation rather than a definitional one, which matters because the two have different remedies — the first is fixable with [[optical-tracking-data|tracking data]], the second is not.

## See Also

- [[expected-possession-value]]
- [[expected-threat]]
- [[vaep]]
- [[martingale-epv]]
- [[expected-goals]]
- [[intent-vs-outcome-valuation]]
- [[player-rating-time-series]]
- [[markov-game]]
- [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]]
- [[football-performance-time-series|Valuing Players Over Time Summary]]
