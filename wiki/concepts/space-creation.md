---
title: "Space Creation"
type: concept
tags: [space-creation, off-ball, sports-analytics, player-evaluation, pitch-control, counterfactual, optical-tracking-data, tactical-analysis]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Space Creation

Off-ball movement that generates opportunity **for a teammate** rather than for oneself. The run that drags a marker away, the decoy that pulls a defensive line, the rotation that opens a passing lane someone else uses.

Distinguished from off-ball positioning by *who benefits*. A striker drifting into a dangerous pocket is positioning himself; a striker dragging a centre-back across so the winger can cut inside is creating space.

## Why It Resists Measurement

Three problems compound, and they are why this was the last major gap in the vault's [[action-valuation]] coverage.

**No event is generated.** The creator never touches the ball. [[event-stream-data|Event data]] records nothing at all — not a bad action, but *no* action. Only [[optical-tracking-data|tracking]] sees it.

**The benefit accrues elsewhere.** Every valuation framework except one credits the player performing the valued act. If the winger scores, the winger is credited; the run that made it possible is invisible in the ledger.

**The counterfactual is unobserved.** To say a run created space, you must say what would have happened without it — and that never happened. This is why the only working approaches are [[counterfactual-baseline|counterfactual]].

## Two Ways to Measure It

**Space as a quantity.** Fernández & Bornn's *Wide Open Spaces* (2018) measures space generated directly, from [[pitch-control|pitch control]] surfaces — how much controllable area a movement opens. This says a player created space without saying whether the space mattered.

**Space as value transferred.** [[c-obso|C-OBSO]] measures the improvement in a *teammate's* scoring chance attributable to the mover deviating from predicted movement:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

where $k$ is the eventual shooter and the primed term uses a [[trajectory-prediction|predicted reference trajectory]] for player $i$. This says nothing about area — only about whether someone else's chance improved.

The second is the more useful framing for [[player-evaluation]], because space in an irrelevant part of the pitch is worth nothing. It is also the more fragile: it inherits the reference model's errors, and is identically zero under perfect prediction.

## The Evidence That It Is Real

The strongest result in this vault for space creation being a distinct, valuable skill comes from what it is compared against. On the same 15 players:

| Metric | ρ with annual salary |
|---|---|
| **C-OBSO** (space created for others) | **0.45** (p = 0.046) |
| [[obso\|OBSO]] (own scoring opportunity) | −0.28 (ns) |
| Goals | −0.23 (ns) |

Neither a player's own off-ball opportunity nor his goal tally relates to what his club pays him. The space he creates for others does.

Salary is heavily confounded — by age, position, nationality, contract timing, reputation — so this is suggestive rather than decisive. But being the only positive result among three metrics tested on one sample makes it harder to dismiss as fishing, and the direction matches what coaches say about players whose value exceeds their output.

## Why Clubs Care

Space creation is the clearest instance of a general recruitment problem: **the players hardest to value are the ones whose contribution is structural rather than terminal.** A target man who occupies two centre-backs, a midfielder whose rotation unlocks a build-up pattern, a winger who holds width to stretch a block — all raise their team's output without appearing in it.

Traditional metrics rate these players by what they do with the ball, which is precisely the part of their game that matters least. That is a market inefficiency in the same family as the [[player-development-curve|age-curve]] one: a systematic mispricing that a club willing to measure differently could exploit.

## Limitations of Current Measurement

- **Scale.** C-OBSO predicts 3 of 22 players for computational reasons; full-squad evaluation is described as prohibitively expensive.
- **Shot-ending sequences only.** Movement that creates chances not converted into shots is invisible.
- **Negative values clipped.** C-OBSO cannot penalise movement that *destroys* space, because negatives are treated as predictor error.
- **No interpretable scale.** Values sit in the 0.001–0.01 range.
- **Defensive space denial is unaddressed** — the mirror-image concept, where a player closes space for the opposition, has no equivalent metric here.

## See Also

- [[c-obso]] · [[obso]] · [[off-ball-value]] · [[counterfactual-baseline]]
- [[pitch-control]] · [[trajectory-prediction]] · [[probability-surface]]
- [[action-valuation]] · [[recruitment]] · [[tactical-analysis]]
- [[masakiyo-teranishi]] · [[william-spearman]] · [[javier-fernandez]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
