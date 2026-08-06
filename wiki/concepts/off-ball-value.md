---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/wide_open_spaces_creation_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md]
confidence: 0.85
provenance:
  extracted: 62%
  inferred: 25%
  generated: 8%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Off-Ball Value

Quantifying the contribution of players who do not have the ball — the runs that stretch a defence, the positioning that opens a passing lane, the shape a defence holds to deny space.

The framing statistic, quoted by both origin papers: **a player has the ball for roughly 3 of 90 minutes.**

Long the largest acknowledged gap here. **Five held mechanisms now address it.**

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**. Fixable only with [[optical-tracking-data|tracking]] — though [[gvdep|GVDEP]] and [[drso|DRSO]] both show partial observation goes further than expected.

## Five Mechanisms

### 1. Read a value surface at player positions — the receiver

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value as a by-product of the [[probability-surface|pass EPV surface]]. [[obso|Spearman's OBSO]] is the same move with a physically-grounded surface.

### 2. Multiply control by value, and difference it over time — the occupier

[[space-occupation-gain|Fernández & Bornn (2018)]] define quality of owned space as $Q_i(t) = PC_i(t)\,V(t)$ — [[pitch-control]] times [[pitch-value-model|pitch value]] — and take **Space Occupation Gain** as the thresholded change in $Q$ over a three-second window.

The distinguishing feature is that it is a **rate rather than a level**. Routes 1 and 5 ask what a player's position is worth *now*; SOG asks what he *gained*, which credits movement rather than standing.

### 3. Put all 22 positions in the model state — the defence

[[vdep]] appends all 22 coordinates sorted by proximity to the ball; [[shap]] confirms these dominate. [[gvdep|GVDEP]] then measured how much was needed: **ball-gain prediction saturates at three or four players**, and the other three targets gain nothing from player positions at all.

That suggests off-ball value for *defensive* purposes is a **local** quantity, carried by the few players nearest the ball rather than by the configuration as a whole.

### 4. Compare against a predicted reference — the creator

[[c-obso]] credits the improvement in *someone else's* [[obso|OBSO]] attributable to deviating from a predicted trajectory. See [[space-creation]].

### 5. Compare against an optimal position — the defender

[[drso|DRSO]] moves the nearest defender to each vertex of his grid cell and takes the position minimising danger. Same [[counterfactual-baseline|counterfactual machinery]] as route 4, opposite reference: deviation from **optimal** rather than from **normal**.

### The mechanisms compared

| | Surface at position | Control × value, differenced | 22 positions in state | Predicted reference | Optimal position |
|---|---|---|---|---|---|
| Values | The **receiver** | The **occupier** | The **defence** | The **creator** | The **defender** |
| Measures | A level | **A rate** | A level | A rate | A level |
| Reported unit | Player | Player | **Team only** | Player | **Team** |
| Example | EPV surface, [[obso]] | [[space-occupation-gain\|SOG]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO; necessity unproven — a Shapley-style attribution would individuate without intervening. rests-on: claim:counterfactual-individuates]

Note that routes 2 and 4 both produce per-player *creation* credit without a counterfactual — [[space-occupation-gain|SGG]] does it by spatial predicate. That does not falsify the claim, since SGG attributes by **co-occurrence** rather than by establishing what would otherwise have happened, but it is the nearest thing to a counter-example the vault holds.

## What Is Now Covered

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, [[obso\|OBSO]] |
| Space gained by movement | **Covered** — [[space-occupation-gain\|SOG]] |
| Team defensive contribution | **Covered** — [[vdep]], [[gvdep]] |
| Credit for creating space for a teammate | **Covered twice** — [[space-occupation-gain\|SGG]] and [[c-obso]], by different mechanisms |
| Per-defender positioning quality | **Computed but not reported** — [[drso\|DRSO]] |
| Movement over time | Partial — C-OBSO 4 s, SOG 3 s windows |
| Errors of omission | Open |

## What Remains Open

- **Per-player defensive results are unpublished.** [[drso|DRSO]] computes $Diff_{opt-obs}$ per named defender; every published result averages to teams.
- **The two space-creation methods have never been compared**, despite being directly comparable on a single match.
- **Errors of omission** — a defender who fails to press generates no event and occupies no notably bad position.
- **Scale.** C-OBSO predicts 3 of 22; SOG/SGG analyse one match.
- **No reliability figure for any off-ball metric here.**^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. Re-checked on the Wide Open Spaces ingest, 2026-07-27: still holds. rests-on: claim:no-reliability-for-off-ball-metrics] Since [[split-half-reliability|reliability]] is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

## What the Evidence Shows

**[[obso|OBSO]] predicts next-match goals better than shots or goals do** — 0.26 against 0.17 and 0.12, at player level against an independent outcome. The strongest [[predictive-validity]] result in the vault.

**[[c-obso]] correlates 0.45 with salary** where own-opportunity (−0.28) and goals (−0.23) do not.

**Occupation and generation are distinct skills**, found independently by [[space-occupation-gain|SOG/SGG]] in La Liga and [[c-obso|C-OBSO]] in the J-League. Two methods, two leagues, same structural finding.

**The gain/attacked trade-off replicates** across [[vdep|VDEP]] and [[gvdep|GVDEP]] — J-League and European tournament, men's and women's football.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** [[expected-value-possession-framework|EPV]] between David Silva and each teammate; [[space-occupation-gain|SGG]]'s generator–receiver matrix does the same for space, showing a reciprocal Suárez↔Messi pair and Busquets receiving from nearly everyone.

**Coaching defensive positioning.** [[drso|DRSO]] outputs a specific alternative position — advice rather than a score.

**Broadcast-only analysis.** [[gvdep|GVDEP]], [[drso|DRSO]] and [[obso|OBSO]] all deliberately minimise data requirements.

## Relation to Pitch Control

Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*. [[obso|OBSO]] and [[space-occupation-gain|SOG]] both make the relationship explicit by multiplying a control surface by a value surface — with **different control models and different value models**, and no comparison between them. See [[pitch-control-traditions-compared]].

## See Also

- [[space-occupation-gain]] · [[c-obso]] · [[obso]] · [[drso]] · [[space-creation]] · [[counterfactual-baseline]]
- [[pitch-control]] · [[pitch-value-model]] · [[probability-surface]] · [[vdep]] · [[gvdep]] · [[defensive-valuation]]
- [[action-valuation]] · [[shap]] · [[trajectory-prediction]] · [[tactical-analysis]] · [[recruitment]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[keisuke-fujii]] · [[rikuhei-umemoto]]
- [[wide-open-spaces-space-creation|Wide Open Spaces]] · [[beyond-expected-goals|Spearman 2018]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] · [[team-defense-positioning-counterfactuals|DRSO]]
