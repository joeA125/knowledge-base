---
title: "Space Creation"
type: concept
tags: [space-creation, off-ball, sports-analytics, player-evaluation, pitch-control, counterfactual, optical-tracking-data, tactical-analysis]
sources: [raw/papers/wide_open_spaces_creation_football.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 28%
  generated: 5%
  imported: 2%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Space Creation

Off-ball movement that generates opportunity **for a teammate** rather than for oneself. The run that drags a marker away, the decoy that pulls a defensive line, the rotation that opens a lane someone else uses.

Distinguished from off-ball positioning by *who benefits*. A striker drifting into a dangerous pocket is positioning himself; a striker dragging a centre-back across so the winger can cut inside is creating space.

## Why It Resists Measurement

**No event is generated.** The creator never touches the ball. [[event-stream-data|Event data]] records nothing at all — not a bad action, but *no* action. Only [[optical-tracking-data|tracking]] sees it.

**The benefit accrues elsewhere.** Most valuation frameworks credit the player performing the valued act. If the winger scores, the winger is credited; the run that made it possible is invisible in the ledger.

**The counterfactual is unobserved.** To say a run created space you must say what would have happened without it — and that never happened.

## Two Held Approaches

> **Superseded, 2026-07-27.** This page previously described the Fernández & Bornn route as cited-but-not-held. [[wide-open-spaces-space-creation|Both papers are now held]], and they turn out to differ more sharply than the earlier framing implied — not in what they measure but in **how they attribute credit**.

### Space as value transferred, by spatial predicate

[[space-occupation-gain|Fernández & Bornn (2018)]] define **Space Generation Gain** as a logical condition on dragging: an opponent starts near teammate $i'$, ends near generator $i$, leaves $i'$, and travelled far enough for the attraction to be real. Where the freed space clears a threshold, the generator is credited.

The value being transferred is quality of owned space, $Q = PC \times V$ — [[pitch-control]] times [[pitch-value-model|pitch value]].

### Space as value transferred, by counterfactual

[[c-obso|Teranishi et al. (2022/23)]] credit the improvement in a teammate's [[obso|OBSO]] attributable to the mover **deviating from a predicted reference trajectory**:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

### The difference that matters

Both answer "how much did this player's movement help a teammate". They differ in **what licenses the attribution**:

| | SGG (2018) | C-OBSO (2022/23) |
|---|---|---|
| Attribution basis | **Co-occurrence** — a defender moved from A to B | **Deviation** — the player moved unlike the model expected |
| Fails when | The defender moved for unrelated reasons | The predictor is wrong |
| Needs a trained model | Only for pitch value | Yes — a trajectory predictor |
| Degenerate case | None | **Identically zero under perfect prediction** |
| External validation | None — one match | Salary correlation 0.45 |

**Neither dominates, and the failure modes are complementary.** The predicate is cheap and transparent but cannot distinguish being *dragged* from happening to move. The counterfactual is principled but inherits its predictor's errors and requires that predictor to be imperfect. See [[counterfactual-baseline]].

An unrun comparison: **do they agree on who creates space?** Both could be computed on the same match, and neither paper cites the other.^[generated: the observation that these two are directly comparable and never compared is drawn here. rests-on: absence:no-held-source-compares-sgg-and-cobso — ⚠️ re-check on any space-creation ingest]

## The Evidence It Is Real

**C-OBSO against salary**, 15 Yokohama players:

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

Neither a player's own off-ball opportunity nor his goal tally relates to what his club pays him. The space he creates for others does.

**SGG separates generation from occupation**, Barcelona–Villarreal 2017. Iniesta, Busquets and Messi lead occupation (41% of team SOG); Neymar and Suárez lead **generation** while sitting mid-table on occupation. Full-backs generate almost nothing — moving toward a touchline drags nobody useful.

Two independent methods, two datasets, two leagues, both finding that **generation and occupation are distinct skills** and that the players strong at one are not the players strong at the other. That convergence is stronger than either result alone.

## Why Clubs Care

Space creation is the clearest instance of a general recruitment problem: **the players hardest to value are the ones whose contribution is structural rather than terminal.** A target man occupying two centre-backs, a midfielder whose rotation unlocks a build-up, a winger holding width to stretch a block — all raise their team's output without appearing in it.

Traditional metrics rate these players by what they do with the ball, which is precisely the part of their game that matters least. A market inefficiency in the same family as the [[player-development-curve|age-curve]] one. See [[recruitment]].

## Limitations of Current Measurement

- **Scale.** C-OBSO predicts 3 of 22 players; SGG analyses one match.
- **Shot-ending sequences only** for C-OBSO, so movement creating chances that never become shots is invisible.
- **Unoccupied generated space is excluded** by SGG — dragging a defender out of a zone nobody enters counts for nothing, which its authors name as a case they chose not to model.
- **Negative values clipped** in C-OBSO, so it cannot penalise movement that *destroys* space.
- **Marking intensity confounds SGG's occupation figures** — Neymar and Suárez score low on SOG, attributed to close marking rather than poor movement.
- **No interpretable scale** for either.
- **No [[split-half-reliability|reliability]] figure** for either.
- **Defensive space denial is unaddressed** — the mirror-image concept, closing space for the opposition, has no equivalent metric here. [[drso|DRSO]] is the nearest thing and asks a different question.

## See Also

- [[space-occupation-gain]] · [[c-obso]] · [[obso]] · [[off-ball-value]] · [[counterfactual-baseline]]
- [[pitch-control]] · [[pitch-value-model]] · [[trajectory-prediction]] · [[probability-surface]] · [[imitation-learning]]
- [[action-valuation]] · [[recruitment]] · [[tactical-analysis]] · [[defensive-valuation]] · [[drso]]
- [[javier-fernandez]] · [[luke-bornn]] · [[masakiyo-teranishi]] · [[william-spearman]]
- [[wide-open-spaces-space-creation|Fernández & Bornn Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
