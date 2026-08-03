---
title: "Off-Ball Value"
type: concept
tags: [off-ball, space-creation, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md]
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

An [[event-stream-data|event stream]] records actions. Off-ball contribution is by definition the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**. Fixable only with [[optical-tracking-data|tracking]] — though [[gvdep|GVDEP]] shows partial tracking goes further than expected.

## Four Routes

### 1. Read a value surface at player positions — the receiver

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value as a by-product of the [[probability-surface|pass EPV surface]]. If the surface gives the EPV of a pass to every location, the value of a player *standing* somewhere is immediately available.

[[obso|Spearman's OBSO]] is the same move with a physically-grounded surface — transition × control × score, read at the receiver's position.

### 2. Put all 22 positions in the model state — the defence

[[football-defence-evaluation-vdep|Toda et al.]]'s [[vdep]] appends all 22 coordinates plus each player's distance from the ball, **sorted by proximity**. [[shap]] confirms these dominate both classifiers.

[[gvdep|GVDEP]] then measured how much of that was needed: **ball-gain prediction saturates at three or four players**, and scores, concedes and being-attacked gain nothing from player positions at all. Most of what VDEP fed its classifier was unnecessary — and for the concedes classifier, actively harmful (F1 0.15 → 0.08 as players are added).

That result matters beyond VDEP. It suggests off-ball value for *defensive* purposes is a **local** quantity, carried by the few players nearest the ball, not by the configuration as a whole.

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
| Example | EPV surface, [[obso\|OBSO]] | [[vdep]], [[gvdep]] | [[c-obso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]], where a Shapley-style objection demotes it from law to tendency. rests-on: claim:counterfactual-individuates] VDEP and C-OBSO use comparable tracking data; VDEP puts everything into one classifier and gets one number per configuration, while C-OBSO intervenes on one named player.

## What Is Now Covered

**Correction, 2026-07-27.** This page previously listed "credit for creating space someone else exploits" among the things no framework captures. Superseded by held sources — an `absence:` claim that expired on ingest.

| Capability | Status |
|---|---|
| Positional value if the ball arrives | **Covered** — EPV surface, [[obso\|OBSO]] |
| Team defensive contribution | **Covered** — [[vdep]], [[gvdep]] |
| **Credit for creating space for a teammate** | **Covered** — [[c-obso]] |
| Individual defensive credit | Cited only — Umemoto & Fujii (**2023**), not held |
| Movement over time | Partial — C-OBSO uses 4 s windows |
| Errors of omission | Open |

> ⚠️ **Two Umemoto papers, easily confused.** [[gvdep|GVDEP]] (Umemoto, Tsutsui & Fujii, **2022**) is held and is **team-level**. The counterfactual-positioning work that would individuate defensive credit is Umemoto & Fujii, **2023**, a different paper, **not held**. The vault previously conflated them and appeared to have closed a gap it had not. See [[defensive-valuation]].

## What Remains Open

- **Individual defensive credit** is addressed in the literature but not in `raw/`, so unverified.
- **Scale.** C-OBSO predicts three of 22 players; the full-squad version is described as prohibitively expensive. GVDEP's result offers a possible reprieve — if three or four players carry the signal, full-squad evaluation may be unnecessary rather than merely expensive.
- **Errors of omission** — a defender who fails to press generates no event and occupies no notably bad position.
- **Interpretable units.** C-OBSO values sit in the 0.001–0.01 range.
- **Negative values.** C-OBSO clips them to zero, so it cannot penalise poor movement.
- **No reliability figure for any off-ball metric here.**^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. Re-checked on the GVDEP ingest, 2026-07-27: still holds. rests-on: claim:no-reliability-for-off-ball-metrics] Since [[split-half-reliability|reliability]] is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

## What the Evidence Shows

**[[obso|OBSO]] predicts next-match goals better than shots or goals do** — 0.26 against 0.17 and 0.12, at player level against an independent outcome. The strongest [[predictive-validity]] result in the vault.

**[[c-obso]] correlates with salary where own-opportunity and goals do not:**

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

Salary is heavily confounded, but being the only positive result among three tested on one sample makes it harder to dismiss.

**The gain/attacked trade-off replicates** across [[vdep|VDEP]] and [[gvdep|GVDEP]] — J-League and European tournament, men's and women's football ($r = -0.757$, $p = 0.001$ in Euro 2020). Teams that recover more concede more territory. One of the few genuine replications in this literature.

## Applications

**Pressing analysis.** Off-ball advantage heatmaps per opponent formation — see [[tactical-analysis]].

**Pairwise relationships.** On-ball and off-ball EPV between David Silva and each teammate, split by direction. [[c-obso]] formalises exactly this relational quantity as a metric.

**Defensive style profiling.** Recovery rate against being-attacked rate.

**Broadcast-only analysis.** [[gvdep|GVDEP]] and [[obso|OBSO]] both deliberately minimise data requirements, making off-ball work available without a tracking licence.

## Relation to Other Approaches

| Approach | Off-ball handling |
|---|---|
| [[expected-threat\|xT]], [[vaep]], [[pass-carry-reward\|PCR]] | None — on-ball events only |
| [[martingale-epv\|Martingale EPV]] | Implicit; tracking state includes all players, no per-player credit |
| **Pass EPV surface / [[obso\|OBSO]]** | **Positional value, attacking, per player** |
| **[[vdep]], [[gvdep]]** | **Defensive contribution, team level** |
| **Umemoto & Fujii (2023)** | **Defensive positioning, per player** — cited, not held |
| [[pitch-control]] | Values *space*, not the value of receiving there |

Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*.

Note the vault holds **two pitch-control traditions**, never compared — see [[pitch-control-traditions-compared]].

## See Also

- [[c-obso]] · [[obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]]
- [[probability-surface]] · [[pitch-control]] · [[vdep]] · [[gvdep]] · [[defensive-valuation]] · [[action-valuation]] · [[shap]]
- [[william-spearman]] · [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[optical-tracking-data]]
- [[action-valuation-frameworks-compared]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
