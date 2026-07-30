---
title: "C-OBSO (Creating Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, space-creation, sports-analytics, player-evaluation, counterfactual, trajectory-prediction, optical-tracking-data, action-valuation, single-source]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.75
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# C-OBSO (Creating Off-Ball Scoring Opportunity)

[[creating-scoring-opportunities-trajectory-prediction|Teranishi, Tsutsui, Takeda & Fujii's]] metric for the player whose **movement improves someone else's** scoring chance. The vault's only framework that assigns value *relationally* — from the mover to the beneficiary — and its only quantitative treatment of [[space-creation]].

## The Construction

For off-ball player $i$ and the eventual ball carrier $k$:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

- $V^k_{OBSO}$ — the shooter's [[obso|OBSO]] in the situation that actually occurred
- $V'^k_{OBSO}$ — the shooter's OBSO in a counterfactual where player $i$ moved as a **[[trajectory-prediction|predicted reference trajectory]]** says an average player would have

So $i$ is credited with *the improvement in $k$'s scoring chance attributable to $i$ deviating from expected movement.* In the worked example, A1 drags his marker further than predicted, opening space for shooter A2; C-OBSO is positive.

Three ingredients: a [[obso|modified OBSO]] with a defender-aware score model, a GVRNN trajectory predictor trained on opponent data via [[imitation-learning]] to generate "league average" reference movement, and the difference between them.

## The Odd Property at Its Centre

**If prediction were perfect, C-OBSO would be identically zero.** The authors state this outright.

The metric measures deviation from a *particular model's* expectation, so it depends on that model being imperfect. Improving the trajectory predictor shrinks the metric. That is an uncomfortable dependency, and it makes C-OBSO less a measurement of a player property than a measurement of *how far a player departs from a learned norm* — a coherent thing to want, but not the same thing.

It also means values are not portable. A C-OBSO computed with a different predictor is a different quantity, and cross-study comparison would be meaningless without fixing the reference model.

## Results

Against annual salary, 15 Yokohama players with ≥10 evaluated sequences:

| Metric | ρ with salary | p |
|---|---|---|
| **C-OBSO** | **0.45** | **0.046** |
| [[obso\|OBSO]] | −0.28 | 0.154 |
| Goals | −0.23 | 0.208 |

The comparison is what carries the argument. **On the same players, neither a player's own off-ball scoring opportunity nor his goal tally relates to salary at all** — only the space he creates for others does. Being the only positive result among three tested on one sample makes it harder to dismiss as fishing.

Two players well above the trend both won individual awards that season and received large pay rises the following year (11M → 40M and 20M → 60M yen).

### The expert-ratings table, read carefully

| Player | ρ (C-OBSO vs rating) | ρ (goals vs rating) |
|---|---|---|
| Nakagawa (season MVP) | **0.75** | 0.63 |
| Marcos | 0.27 (ns) | 0.71 |
| Edigar | −0.37 (ns) | 0.91 |

**Goals predict expert match ratings strongly for everyone; C-OBSO predicts them only for the MVP.** Seven further players show no correlation.

The generous reading is that ratings are largely goal-driven and C-OBSO measures something raters mostly ignore — except for one player whose reputation rested on more than finishing. The sceptical reading is that one significant result in eight is what chance produces. The paper reports both the positive and null results without over-claiming; the evidence does not settle it.

## Limitations

- **Negative values are clipped to zero.** Justified because the *predicted defender* often fails to take a sensible position, so negatives reflect predictor error rather than player error. But the metric therefore cannot penalise bad movement, and every player's mean is inflated by a floor.
- **Three of 22 players predicted**, for computational reasons. Fujii has described the full-squad version as prohibitively expensive.
- **Values are tiny** (0.001–0.01) and on no interpretable scale. Named as future work.
- **One team, 34 games, one season.**
- **Salary is heavily confounded** by age, position, nationality, contract timing and reputation.
- **No [[split-half-reliability|reliability]] figure** — universal in this literature.
- **Only shot-ending sequences** are evaluated (412 of them), so movement that creates chances not converted into shots is invisible.

## Where It Sits

| | [[obso]] | **C-OBSO** | [[vdep]] | [[probability-surface\|Pass EPV surface]] |
|---|---|---|---|---|
| Values | The receiver | **The creator** | The defence | The receiver's location |
| Unit | Player | **Player** | Team | Player |
| Perspective | Attack | **Attack** | Defence | Attack |
| Mechanism | Surface read at position | **Counterfactual difference** | Classifier on 22 positions | Surface read at position |

C-OBSO and [[vdep]] are the two Fujii-group answers to off-ball valuation, taking opposite routes — VDEP puts everything in the model state and gets a team number; C-OBSO intervenes on one player and gets an individual number. The individuating ingredient in both cases is **the counterfactual**, not the data. See [[counterfactual-baseline]].

## See Also

- [[obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[imitation-learning]]
- [[off-ball-value]] · [[vdep]] · [[defensive-valuation]] · [[pitch-control]] · [[action-valuation]]
- [[masakiyo-teranishi]] · [[keisuke-fujii]] · [[william-spearman]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
