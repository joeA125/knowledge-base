---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Off-Ball Value

Quantifying the contribution of players who do not have the ball — the runs that stretch a defence, the positioning that opens a passing lane, the shape a defence holds to deny space.

The framing statistic: **a player has the ball for roughly 3 of 90 minutes.** Everything else is off-ball, continuous, and — until recently — invisible to this vault's frameworks.

Long the largest acknowledged gap here. **Three held sources now address it**, by three different mechanisms.

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**.

No modelling sophistication recovers this from event data. It is a data limitation, fixable only with [[optical-tracking-data|tracking]].

## Three Routes

### 1. Read a value surface at player positions — attacking, the receiver

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value as a by-product of the [[probability-surface|pass EPV surface]]. If the surface gives the EPV of a pass to every location, the value of a player *standing* somewhere is immediately available.

[[obso|Spearman's OBSO]] is the same move with a different surface — control × transition × score, read at the receiver's position.

No model of off-ball behaviour is required. Neither predicts runs nor credits movement; both compute what locations are worth and read off who occupies them.

### 2. Put all 22 positions in the model state — defending, the team

[[football-defence-evaluation-vdep|Toda et al.]]'s [[vdep]] appends all 22 coordinates plus each player's distance from the ball, **sorted by proximity** — a cheap permutation-invariance trick that gives "the nearest defender" a fixed feature slot. [[shap]] confirms these dominate both classifiers.

### 3. Compare against a predicted reference — attacking, the *creator*

[[creating-scoring-opportunities-trajectory-prediction|Teranishi, Tsutsui, Takeda & Fujii]]'s [[c-obso]] credits an off-ball player with the improvement in *someone else's* [[obso|OBSO]] attributable to his deviating from a predicted "league average" trajectory:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

This is the only framework here that assigns value **relationally** — from the mover to the beneficiary.

### The three compared

| | Surface at position | 22 positions in state | Predicted reference |
|---|---|---|---|
| Values | The **receiver** | The **defence** | The **creator** |
| Output unit | Player | **Team only** | Player |
| Mechanism | Read a surface | Model input | [[counterfactual-baseline\|Counterfactual difference]] |
| Cost | Substantial | Modest | Substantial |
| Example | EPV surface, [[obso\|OBSO]] | [[vdep]] | [[c-obso]] |

**The individuating ingredient is the counterfactual, not the data.** VDEP and C-OBSO use comparable tracking data; VDEP puts everything into one classifier and gets one number for the configuration with no principled way to split it, while C-OBSO intervenes on one named player and gets his number. Once you ask "what if *this* player had moved differently", credit is unambiguous.

## What Is Now Covered

**Correction, 2026-07-27.** This page previously listed "credit for creating space someone else exploits" among the things no framework captures, and described combining the spatial and defensive routes as an unattempted opportunity. Both claims are now superseded by held sources.

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, OBSO |
| Team defensive contribution | **Covered** — [[vdep]] |
| **Credit for creating space for a teammate** | **Covered** — [[c-obso]] |
| Individual defensive credit | Cited only — Umemoto & Fujii (2023), not held |
| Movement over time | Partial — C-OBSO uses 4 s windows |
| Errors of omission | Open |

## What Remains Open

- **Individual defensive credit** is addressed in the literature (Umemoto & Fujii, 2023, counterfactual positioning) but not in `raw/`, so unverified here. See [[defensive-valuation]].
- **Scale.** C-OBSO predicts three of 22 players, for computational reasons; Fujii describes the full-squad version as prohibitively expensive. A method whose appeal is modelling interaction is restricted in practice to the smallest interacting subset.
- **Errors of omission** — a defender who fails to press generates no event and occupies no notably bad position.
- **Interpretable units.** C-OBSO values sit in the 0.001–0.01 range on no meaningful scale, which its authors name as future work.
- **Negative values.** C-OBSO clips them to zero, so it cannot penalise poor movement.
- **No [[split-half-reliability|reliability]] figure** for any off-ball metric here.

## What the Evidence Shows

The strongest single result is C-OBSO's, because of what it is compared against. On the same 15 players:

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

Neither a player's own off-ball opportunity nor his goal tally relates to what his club pays him; the space he creates for others does. Salary is heavily confounded — by age, position, nationality, contract timing and reputation — but being the only positive result among three tested on one sample makes it harder to dismiss.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** On-ball and off-ball EPV between David Silva and each teammate, split by direction, distinguishing those who *create space for* him from those who *benefit from* his passing. C-OBSO formalises exactly this relational quantity as a metric.

**Defensive style profiling.** Recovery rate against being-attacked rate.

**Player valuation.** C-OBSO is the only off-ball metric here shown to relate to an external measure of player worth.

## Relation to Other Approaches

[[pitch-control]] and off-ball value are complementary and often confused. Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for.

Note the vault now holds **two pitch-control traditions** — Spearman's arrival-time Poisson model (inside OBSO) and Fernández & Bornn's Gaussian influence model — used as inputs to different value models and never compared. See [[william-spearman]].

## See Also

- [[c-obso]] · [[obso]] · [[counterfactual-baseline]] · [[trajectory-prediction]]
- [[probability-surface]] · [[pitch-control]] · [[vdep]] · [[defensive-valuation]]
- [[action-valuation]] · [[shap]] · [[tactical-analysis]] · [[optical-tracking-data]]
- [[william-spearman]] · [[keisuke-fujii]] · [[masakiyo-teranishi]]
- [[expected-value-possession-framework|Soccer EPV Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
