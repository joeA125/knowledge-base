---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, reinforcement-learning, multi-agent, construct-validity, evaluation]
sources: [raw/papers/wide_open_spaces_creation_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 25%
  generated: 10%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Off-Ball Value

Quantifying the contribution of players who do not have the ball — the runs that stretch a defence, the positioning that opens a passing lane, the shape a defence holds to deny space.

The framing statistic, quoted by three origin papers: **a player has the ball for roughly 3 of 90 minutes.** [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] give the complement — approximately 87 of 90 minutes off the ball.

Long the largest acknowledged gap here. **Six held mechanisms now address it.**

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**. Fixable only with [[optical-tracking-data|tracking]] — though [[gvdep|GVDEP]] and [[drso|DRSO]] both show partial observation goes further than expected.

## Six Mechanisms

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

### 6. Learn an action-value over every player and timestep — the mover

> **Added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]].

Instantiate **one [[multi-agent-reinforcement-learning|RL agent per player]] and learn $Q(s,a)$ over a 14-action space** by [[temporal-difference-learning|temporal difference]]. Off-ball value is the Q-value of a movement direction; a player's contribution is the aggregate over the possession.

**This is the only route that requires no reference to difference against.** Routes 4 and 5 need a counterfactual baseline; routes 1 and 2 need a value surface; route 3 needs a classifier target. Here the counterfactual comes free, because a learned $Q$ is *defined* over actions nobody took.

The cost is that the unchosen actions' values are constrained only by [[action-supervision]] and network smoothness — SARSA never targets them. So "free" means "not separately estimated", not "well estimated".

### The mechanisms compared

| | Surface at position | Control × value, differenced | 22 positions in state | Predicted reference | Optimal position | **Learned $Q$** |
|---|---|---|---|---|---|---|
| Values | The **receiver** | The **occupier** | The **defence** | The **creator** | The **defender** | **The mover** |
| Measures | A level | **A rate** | A level | A rate | A level | **A rate** |
| Needs a baseline | Yes (surface) | Yes (surface) | No | **Yes** | **Yes** | **No** |
| Reported unit | Player | Player | **Team only** | Player | **Team** | **Player** |
| Example | EPV surface, [[obso]] | [[space-occupation-gain\|SOG]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] | Nakahara et al. |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO; **weakened 2026-08-07** — route 6 individuates via per-agent value functions rather than by intervening, which is a second non-counterfactual individuator alongside SGG. rests-on: claim:counterfactual-individuates]

Routes 2 and 4 both produce per-player *creation* credit without a counterfactual — [[space-occupation-gain|SGG]] does it by spatial predicate. Route 6 does it by **agent decomposition**. Neither falsifies the claim outright, since SGG attributes by co-occurrence and route 6's per-agent split is itself a modelling choice rather than a measurement, but two independent non-counterfactual routes is materially more pressure than one.

## The Two Off-Ball Metrics That Have Been Compared Disagree

> **Added 2026-08-07 — the first head-to-head in this vault, and it is not reassuring.**

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] correlate their Q-values against the other off-ball metrics available on **the same club, season and data provider**:

| Against | $\rho$ | Reading |
|---|---|---|
| [[obso\|OBSO]] | −0.305 | Low negative |
| [[c-obso\|C-OBSO]] | **0.182** | **No relationship** |
| Season goals | −0.761 | Strong negative |
| Expert ratings | −0.218 | Low negative |

C-OBSO and the Q-values are both from the [[keisuke-fujii|Fujii group]], both on Yokohama F. Marinos in J1 2019 with [[data-stadium|Data Stadium]] tracking, both presented as measures of off-ball contribution. They are unrelated.

The paper's explanation — C-OBSO ranks forwards, Q-values rank midfielders and defenders — is plausible and partly self-defeating. **If each metric ranks a different position group, neither measures off-ball contribution as such.** The honest report would name the construct more narrowly, or report it as position-conditional.

$N = 14$, so this is weak evidence for any conclusion. But it is the only evidence the field has produced, and it points the wrong way. See [[construct-validity]].

## What Is Now Covered

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, [[obso\|OBSO]] |
| Space gained by movement | **Covered** — [[space-occupation-gain\|SOG]] |
| Team defensive contribution | **Covered** — [[vdep]], [[gvdep]] |
| Credit for creating space for a teammate | **Covered twice** — [[space-occupation-gain\|SGG]] and [[c-obso]], by different mechanisms |
| Per-defender positioning quality | **Computed but not reported** — [[drso\|DRSO]] |
| **Value of a movement not made** | **Covered** — route 6, via $Q$ over unchosen actions |
| Movement over time | Partial — C-OBSO 4 s, SOG 3 s windows; **route 6 covers whole possessions to 30 s** |
| Errors of omission | **Partially covered** — route 6 assigns a value to every available action, so failing to make a valuable run is now visible in principle. Not reported as such by any source |
| **Agreement between the metrics** | **Now measured, and poor** |

## What Remains Open

- **The metrics do not agree.** See above. This displaces "not enough mechanisms" as the field's main problem.
- **Per-player defensive results are unpublished.** [[drso|DRSO]] computes $Diff_{opt-obs}$ per named defender; every published result averages to teams.
- **The two space-creation methods have never been compared**^[generated: the observation that SGG and C-OBSO are directly comparable and never compared is drawn here. **Re-checked 2026-08-07** on ingest of Nakahara et al.: still holds. That paper compares Q-values against C-OBSO and OBSO but not against SGG. rests-on: absence:no-held-source-compares-sgg-and-cobso — ⚠️ re-check on any space-creation ingest], despite being directly comparable on a single match.
- **Errors of omission** — route 6 addresses this in principle; no source reports it.
- **Scale.** C-OBSO predicts 3 of 22; SOG/SGG analyse one match; Nakahara et al. use 14 players from one club, attacking third only.
- **No reliability figure for any off-ball metric here.**^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. **Re-checked 2026-08-07** on the Nakahara ingest: still holds — that paper reports no reliability figure either, and its authors state no ground truth exists. Six mechanisms, zero reliability estimates. rests-on: claim:no-reliability-for-off-ball-metrics] Since [[split-half-reliability|reliability]] is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

**The reliability gap and the disagreement gap are the same gap.** If two metrics disagree, either they measure different things or at least one is unstable — and without a reliability figure those cannot be told apart. That makes reliability the cheapest next measurement in this whole area.

## What the Evidence Shows

**[[obso|OBSO]] predicts next-match goals better than shots or goals do** — 0.26 against 0.17 and 0.12, at player level against an independent outcome. The strongest [[predictive-validity]] result in the vault, and still the only one against an external criterion.

**[[c-obso|C-OBSO]] correlates 0.45 with salary** where own-opportunity (−0.28) and goals (−0.23) do not.

**Occupation and generation are distinct skills**, found independently by [[space-occupation-gain|SOG/SGG]] in La Liga and [[c-obso|C-OBSO]] in the J-League. Two methods, two leagues, same structural finding.

**Off-ball and on-ball ability correlate moderately within a player** — Nakahara et al. report $\rho = 0.618$ between a player's mean on-ball and off-ball Q-values, suggesting a consistent style across both states rather than two separable skills. Mild tension with the occupation/generation split above, and unresolved.

**The gain/attacked trade-off replicates** across [[vdep|VDEP]] and [[gvdep|GVDEP]] — J-League and European tournament, men's and women's football.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** [[expected-value-possession-framework|EPV]] between David Silva and each teammate; [[space-occupation-gain|SGG]]'s generator–receiver matrix does the same for space, showing a reciprocal Suárez↔Messi pair and Busquets receiving from nearly everyone.

**Coaching defensive positioning.** [[drso|DRSO]] outputs a specific alternative position — advice rather than a score.

**Valuing the run not made.** Route 6's per-action Q-values are, in principle, prescriptive at every timestep for every player. No source exploits this; it is the most obvious unclaimed application in this area.

**Broadcast-only analysis.** [[gvdep|GVDEP]], [[drso|DRSO]] and [[obso|OBSO]] all deliberately minimise data requirements. Route 6 does not — it needs full tracking.

## Relation to Pitch Control

Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*. [[obso|OBSO]] and [[space-occupation-gain|SOG]] both make the relationship explicit by multiplying a control surface by a value surface — with **different control models and different value models**, and no comparison between them. See [[pitch-control-traditions-compared]].

Route 6 is the first mechanism here that uses **no pitch-control model at all** — raw positions and velocities go into a [[gated-recurrent-unit|GRU]] and value comes out. Whether that is an advantage (fewer assumptions) or a loss (no [[theory-based-modelling|physical structure]]) is exactly the question [[handcrafted-features-rule]] asks.

## See Also

- [[space-occupation-gain]] · [[c-obso]] · [[obso]] · [[drso]] · [[space-creation]] · [[counterfactual-baseline]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[temporal-difference-learning]] · [[reinforcement-learning]]
- [[pitch-control]] · [[pitch-value-model]] · [[probability-surface]] · [[vdep]] · [[gvdep]] · [[defensive-valuation]]
- [[action-valuation]] · [[shap]] · [[trajectory-prediction]] · [[tactical-analysis]] · [[recruitment]] · [[construct-validity]] · [[split-half-reliability]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[hiroshi-nakahara]]
- [[wide-open-spaces-space-creation|Wide Open Spaces]] · [[beyond-expected-goals|Spearman 2018]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] · [[team-defense-positioning-counterfactuals|DRSO]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]]
