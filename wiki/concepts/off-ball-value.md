---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 25%
  generated: 10%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Off-Ball Value

Quantifying the contribution of players who do not have the ball — the runs that stretch a defence, the positioning that opens a passing lane, the shape a defence holds to deny space.

The framing statistic: **a player has the ball for roughly 3 of 90 minutes.**

Long the largest acknowledged gap here. **Five held mechanisms now address it.**

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**. Fixable only with [[optical-tracking-data|tracking]] — though [[gvdep|GVDEP]] and [[drso|DRSO]] both show partial observation goes further than expected.

## Four Routes

### 1. Read a value surface at player positions — the receiver

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value as a by-product of the [[probability-surface|pass EPV surface]]. [[obso|Spearman's OBSO]] is the same move with a physically-grounded surface — transition × control × score, read at the receiver's position.

### 2. Put all 22 positions in the model state — the defence

[[vdep]] appends all 22 coordinates plus distances from the ball, **sorted by proximity**. [[shap]] confirms these dominate both classifiers.

[[gvdep|GVDEP]] then measured how much was needed: **ball-gain prediction saturates at three or four players**, and scores, concedes and being-attacked gain nothing from player positions at all. For the concedes classifier extra players are actively harmful (F1 0.15 → 0.08).

That suggests off-ball value for *defensive* purposes is a **local** quantity, carried by the few players nearest the ball rather than by the configuration as a whole.

### 3. Compare against a predicted reference — the *creator*

[[c-obso]] credits an off-ball player with the improvement in *someone else's* [[obso|OBSO]] attributable to his deviating from a predicted "league average" trajectory. The only framework here that assigns value **relationally** — from the mover to the beneficiary.

### 4. Compare against an optimal position — the *defender*

[[drso|DRSO]] finds the most dangerous point on the pitch, moves the nearest defender to each vertex of his grid cell, and takes the position minimising danger. $Diff_{opt-obs}$ is how far his actual position was from that optimum.

Same [[counterfactual-baseline|counterfactual machinery]] as route 3, opposite reference: **deviation from optimal** rather than deviation from normal.

### The routes compared

| | Surface at position | 22 positions in state | Predicted reference | Optimal position |
|---|---|---|---|---|
| Values | The **receiver** | The **defence** | The **creator** | The **defender** |
| Reported unit | Player | **Team only** | Player | **Team** (mechanism is per-player) |
| Mechanism | Read a surface | Model input | Counterfactual on trajectory | Counterfactual on position |
| Example | EPV surface, [[obso\|OBSO]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO: same group, comparable data, and intervening on one named defender produces his own number where VDEP and GVDEP stay at team level. Necessity remains unproven — a Shapley-style attribution over agents would individuate without intervening. rests-on: claim:counterfactual-individuates]

## What Is Now Covered

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, [[obso\|OBSO]] |
| Team defensive contribution | **Covered** — [[vdep]], [[gvdep]] |
| Credit for creating space for a teammate | **Covered** — [[c-obso]] |
| **Per-defender positioning quality** | **Computed but not reported** — [[drso\|DRSO]] |
| Movement over time | Partial — C-OBSO uses 4 s windows |
| Errors of omission | Open |

> **Superseded, 2026-07-27.** This row previously read "Individual defensive credit — cited only, not held". [[team-defense-positioning-counterfactuals|Umemoto & Fujii (2023)]] is now held. **DRSO computes $Diff_{opt-obs}$ for each named defender** — but every published result averages three defenders, then events, then teams. The gap is one aggregation step, not a missing method. See [[defensive-valuation]].

## What Remains Open

- **Per-player defensive results are unpublished**, as above.
- **Reachability unchecked.** DRSO never tests whether the defender could have reached the optimal vertex.
- **Possession-share confounds DRSO.** Manchester City conceded fewest and scored poorly, because holding 60%+ possession means rare but dangerous defensive moments.
- **Goalkeeping excluded** from GVDEP and DRSO, which explains two of DRSO's own anomalies.
- **Scale.** C-OBSO predicts three of 22 players; GVDEP suggests three or four may suffice, so this may be less binding than assumed.
- **Errors of omission** — a defender who fails to press generates no event and occupies no notably bad position.
- **No reliability figure for any off-ball metric here.**^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. Re-checked on the DRSO ingest, 2026-07-27: still holds. rests-on: claim:no-reliability-for-off-ball-metrics]

## What the Evidence Shows

**[[obso|OBSO]] predicts next-match goals better than shots or goals do** — 0.26 against 0.17 and 0.12, at player level against an independent outcome. The strongest [[predictive-validity]] result in the vault.

**[[c-obso]] correlates with salary where own-opportunity and goals do not:**

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

**The gain/attacked trade-off replicates** across [[vdep|VDEP]] and [[gvdep|GVDEP]] — J-League and European tournament, men's and women's football.

**[[drso|DRSO]] tracks relegation and season-on-season change**, though with the possession confound above: Leeds worst and relegated; Arsenal, Leeds and Manchester United improving in 2022-23 alongside fewer concedes.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** On-ball and off-ball EPV between David Silva and each teammate, split by direction. [[c-obso]] formalises this relational quantity as a metric.

**Coaching defensive positioning.** [[drso|DRSO]] outputs a specific alternative position, which is advice rather than a score — the only framework here that does.

**Broadcast-only analysis.** [[gvdep|GVDEP]], [[drso|DRSO]] and [[obso|OBSO]] all deliberately minimise data requirements.

## Relation to Other Approaches

| Approach | Off-ball handling |
|---|---|
| [[expected-threat\|xT]], [[vaep]], [[pass-carry-reward\|PCR]] | None — on-ball events only |
| [[martingale-epv\|Martingale EPV]] | Implicit; no per-player credit |
| **Pass EPV surface / [[obso\|OBSO]]** | **Positional value, attacking, per player** |
| **[[vdep]], [[gvdep]]** | **Defensive contribution, team level** |
| **[[c-obso]], [[drso]]** | **Counterfactual credit, per player computed** |
| [[pitch-control]] | Values *space*, not the value of receiving there |

Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*.

The vault holds **two pitch-control traditions**, never compared — see [[pitch-control-traditions-compared]].

## See Also

- [[c-obso]] · [[drso]] · [[obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]]
- [[probability-surface]] · [[pitch-control]] · [[vdep]] · [[gvdep]] · [[defensive-valuation]] · [[action-valuation]] · [[shap]]
- [[william-spearman]] · [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[optical-tracking-data]]
- [[action-valuation-frameworks-compared]]
- [[beyond-expected-goals|Spearman Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
