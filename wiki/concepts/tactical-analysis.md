---
title: "Tactical Analysis"
type: concept
tags: [tactical-analysis, sports-analytics, optical-tracking-data, off-ball, pitch-control, clustering, interpretability, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.75
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Tactical Analysis

Analysis at the level of **team shape and collective behaviour** rather than individual actions or players: how a team builds up, how it presses, which formations trouble it, where it creates and concedes advantage.

Distinct from the vault's other football tasks in its unit of analysis. [[action-valuation]] values actions, [[player-rating-time-series|rating work]] values players, [[football-event-sequences-point-process-mixture|clustering]] characterises possessions. Tactical analysis asks about the team as a configured whole.

## Why It Sits Awkwardly in the Literature

Most football analytics produces **numbers about people**, because that is what the transfer market pays for. Tactical questions — should we press 4-3-3 or 4-2-4 against this opponent? — are the ones coaches ask most often and the ones analytics has served worst.

The obstacle is representational. A tactic is a *relationship among positions over time*, and the standard event-stream representation discards exactly that: a list of on-ball actions cannot express that the defensive line sat ten metres deeper than usual. Progress here has tracked the availability of [[optical-tracking-data|tracking data]] rather than modelling sophistication.

## Two Approaches in the Vault

**Value surfaces conditioned on shape.** [[expected-value-possession-framework|Fernández, Bornn & Cervone]] compute [[off-ball-value|off-ball]] and on-ball EPV-added heatmaps for each formation an opponent used, differenced against the season mean. Applied to Liverpool's 2014/15 buildup:

| Press | Effect on Liverpool |
|---|---|
| 3-4-3 | Off-ball advantage before the second line; breaks the first line centrally |
| 4-4-2 | First line harder to break, but space opens between defence and midfield for long balls wide |
| **4-3-3** | Best at forcing play outside the block |
| 4-2-4 | Most effective at denying the inside, but concedes space beside the midfielders |
| 5-3-2 | Concedes both centrally and in behind |

The conclusion — 4-3-3 unless you have wing-backs quick enough to press wide receptions, in which case 4-2-4 — is the kind of conditional, personnel-dependent recommendation coaches actually work with.

**Possession clustering.** [[football-event-sequences-point-process-mixture|Amezouwui et al.]] approach the same level from the other direction, clustering whole possessions into tactical types (direct counter-attack through to elaborate positional play) without any notion of value.

The two are complementary: clustering says *what kinds of thing a team does*, value surfaces say *where doing them pays*.

## What Makes It Tractable

Three ingredients, all from tracking data:

1. **A shape representation** — [[dynamic-pressure-lines]], giving formation and relative position continuously rather than as a pre-match label.
2. **A spatial value function** — a [[probability-surface]], so advantage can be located rather than merely totalled.
3. **Aggregation across comparable situations** — restricting to organised buildups, then differencing against a baseline so what shows up is *this opponent's* effect rather than general pitch structure.

The differencing step is the one most easily skipped and most necessary. Raw advantage heatmaps look much the same for every team, because they mostly reflect where football happens.

## The Validation Problem, in Its Sharpest Form

Tactical analysis has the weakest validation of anything in this vault, and the reason is structural rather than negligent.

The claim "4-3-3 is the better press against Liverpool" is causal and counterfactual: it asserts what would have happened under a formation that was not used. Observational data contains formations that *were* chosen, by coaches with reasons — a [[selection-bias|selection]] problem exactly parallel to the transfer case in [[positive-unlabeled-learning]]. Teams that pressed Liverpool 4-2-4 may have differed systematically from those that did not.

Nor is there a natural criterion. A valuation metric can be tested for [[split-half-reliability|reliability]] or [[predictive-validity|predictive validity]]; a tactical recommendation would need a team to actually adopt it, in enough matches, against a comparable baseline. No source here does this, and the analyses are presented as illustrative rather than tested.

This is worth stating plainly because tactical output is unusually **persuasive** — heatmaps overlaid on a pitch read as evidence in a way a correlation table does not. The visual quality of the argument is not evidence about its correctness.

## See Also

- [[dynamic-pressure-lines]] · [[pitch-control]] · [[off-ball-value]]
- [[probability-surface]] · [[expected-possession-value]]
- [[football-event-sequences-point-process-mixture|Possession Clustering Summary]]
- [[optical-tracking-data]] · [[selection-bias]] · [[predictive-validity]]
- [[action-valuation-frameworks-compared]]
- [[expected-value-possession-framework|Source Summary]]
