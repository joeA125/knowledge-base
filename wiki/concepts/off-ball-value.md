---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 30%
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

Long the largest acknowledged gap here. **Four held mechanisms now address it.**

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**. Fixable only with [[optical-tracking-data|tracking]].

## Four Routes

### 1. Read a value surface at player positions — the receiver

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value as a by-product of the [[probability-surface|pass EPV surface]]. If the surface gives the EPV of a pass to every location, the value of a player *standing* somewhere is immediately available.

[[obso|Spearman's OBSO]] is the same move with a different, physically-grounded surface — transition × control × score, read at the receiver's position.

### 2. Put all 22 positions in the model state — the defence

[[football-defence-evaluation-vdep|Toda et al.]]'s [[vdep]] appends all 22 coordinates plus each player's distance from the ball, **sorted by proximity** — a cheap permutation-invariance trick. [[shap]] confirms these dominate both classifiers.

### 3. Compare against a predicted reference — the *creator*

[[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]]'s [[c-obso]] credits an off-ball player with the improvement in *someone else's* [[obso|OBSO]] attributable to his deviating from a predicted "league average" trajectory:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

The only framework here that assigns value **relationally** — from the mover to the beneficiary.

### The routes compared

| | Surface at position | 22 positions in state | Predicted reference |
|---|---|---|---|
| Values | The **receiver** | The **defence** | The **creator** |
| Output unit | Player | **Team only** | Player |
| Mechanism | Read a surface | Model input | [[counterfactual-baseline\|Counterfactual difference]] |
| Cost | Substantial | Modest | Substantial |
| Example | EPV surface, [[obso\|OBSO]] | [[vdep]] | [[c-obso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]], where a Shapley-style objection demotes it from law to tendency. rests-on: claim:counterfactual-individuates] VDEP and C-OBSO use comparable tracking data; VDEP puts everything into one classifier and gets one number per configuration, while C-OBSO intervenes on one named player.

## What Is Now Covered

**Correction, 2026-07-27.** This page previously listed "credit for creating space someone else exploits" among the things no framework captures. Superseded by held sources — an `absence:` claim that expired on ingest.

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, [[obso\|OBSO]] |
| Team defensive contribution | **Covered** — [[vdep]] |
| **Credit for creating space for a teammate** | **Covered** — [[c-obso]] |
| Individual defensive credit | Cited only — Umemoto & Fujii (2023), not held |
| Movement over time | Partial — C-OBSO uses 4 s windows |
| Errors of omission | Open |

## What Remains Open

- **Individual defensive credit** is addressed in the literature but not in `raw/`. See [[defensive-valuation]].
- **Scale.** C-OBSO predicts three of 22 players; the full-squad version is described as prohibitively expensive.
- **Errors of omission** — a defender who fails to press generates no event and occupies no notably bad position.
- **Interpretable units.** C-OBSO values sit in the 0.001–0.01 range.
- **Negative values.** C-OBSO clips them to zero, so it cannot penalise poor movement.
- **No reliability figure for any off-ball metric here.**^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. rests-on: claim:no-reliability-for-off-ball-metrics] Since [[split-half-reliability|reliability]] is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

## What the Evidence Shows

**[[obso|OBSO]] predicts next-match goals better than shots or goals do** — 0.26 against 0.17 and 0.12, at player level against an independent outcome. The strongest [[predictive-validity]] result in the vault.

**[[c-obso]] correlates with salary where own-opportunity and goals do not:**

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

Salary is heavily confounded, but being the only positive result among three tested on one sample makes it harder to dismiss.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** On-ball and off-ball EPV between David Silva and each teammate, split by direction, distinguishing those who *create space for* him from those who *benefit from* his passing. [[c-obso]] formalises exactly this relational quantity as a metric.

**Defensive style profiling.** Recovery rate against being-attacked rate.

**Player valuation.** OBSO and C-OBSO are the only off-ball metrics here shown to track an external measure of worth.

## Relation to Other Approaches

| Approach | Off-ball handling |
|---|---|
| [[expected-threat\|xT]], [[vaep]], [[pass-carry-reward\|PCR]] | None — on-ball events only |
| [[martingale-epv\|Martingale EPV]] | Implicit; tracking state includes all players, no per-player credit |
| **Pass EPV surface / [[obso\|OBSO]]** | **Positional value, attacking, per player** |
| **[[vdep]]** | **Defensive contribution, team level** |
| **Umemoto & Fujii (2023)** | **Defensive positioning, per player** — cited, not held |
| [[pitch-control]] | Values *space*, not the value of receiving there |

Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for.

Note the vault holds **two pitch-control traditions**, never compared — see [[pitch-control-traditions-compared]].

## See Also

- [[c-obso]] · [[obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]]
- [[probability-surface]] · [[pitch-control]] · [[vdep]] · [[defensive-valuation]] · [[action-valuation]] · [[shap]]
- [[william-spearman]] · [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[optical-tracking-data]] · [[tactical-analysis]]
- [[action-valuation-frameworks-compared]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
