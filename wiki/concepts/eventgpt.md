---
title: "EventGPT"
type: concept
tags: [sports-analytics, counterfactual, transformer, event-prediction, player-evaluation, entity-embedding, action-valuation, generative-model]
sources: [raw/papers/eventgpt-player-impact-from-team-action-sequences.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# EventGPT

EventGPT ([[eventgpt-player-impact-team-action-sequences|Lee, Hong et al., 2025]]) is a player-conditioned, value-aware next-event prediction model built on a nanoGPT decoder-only [[transformer]]. It is the direct predecessor to [[scoutgpt]] from the same group, and established the design pattern both share.

## Two Ideas

### Player conditioning
Every event token is predicted conditional on **who is acting**, via a learned player embedding. This is what earlier models in the [[large-event-model]] line lack — [[seq2event]], [[nmstpp]] and LEM describe how play unfolds in aggregate, not how a *specific* player alters it.

Crucially, player identity **conditions but is never predicted**. This is what makes [[counterfactual-simulation|counterfactual substitution]] controlled: swap the identity token, hold everything else fixed, and the model cannot regenerate the player it expected.

### Value as a token
The prediction target includes [[on-ball-value|residual OBV]] — expected cumulative future value to the end of the episode — as the **seventh token of each event**, discretised into the shared vocabulary:

$$v_t = (h_t, e_t, x_t, y_t, \delta_t, o_t, rOBV_t)$$

Value is therefore generated rather than predicted by a separate head, which means it is learned jointly with the sequence dynamics rather than layered on afterwards.

## Architecture

nanoGPT decoder-only transformer, with all attributes — categorical, spatial, temporal and value — sharing **one vocabulary and one embedding matrix**, and output projection weights **tied** to the input embedding.

The authors argue this against multi-branch designs that separately embed spatial, temporal and categorical features: a unified vocabulary lets the transformer learn attribute interactions through [[attention-mechanism|attention]] rather than through hand-designed fusion. Trained with [[teacher-forcing]].

Uses [[spadl]] event representation — the only model in this vault's football cluster to do so, where [[nmstpp]] uses WyScout and [[scoutgpt]] uses VERSA. Episodes are defined by *unchanged personnel*, breaking on substitutions and dismissals as well as stoppages, which guarantees a stable context for player conditioning.

## Results

Against LEM and an [[nmstpp]]-derived LEM Transformer baseline: best on event type (82.91%), spatial error roughly halved ($x$ MAE 7.15 → 4.30), timing, and action success. It **loses on rOBV error** (0.009 vs 0.008) and on team-indicator accuracy.

The event-type gain is attributed to player conditioning — knowing who has the ball genuinely improves prediction of what they will do.

## The Case Studies Carry the Argument

The headline metrics are incremental; the counterfactual studies are where the contribution lies.

**Striker fit reverses across systems.** Haaland's predicted rOBV falls from 2.71 in Manchester City's structured build-up to 1.37 in Manchester United's transition-heavy context. Isak scores highest in both, which the authors read as adaptability — generating value without depending on heavy service.

**Stylistic similarity ≠ contextual fit.** Retrieving candidates for Saka's role by embedding similarity surfaces Salah (19.78) and Madueke (19.36) as strong fits, but also Mbeumo (11.80) and Bowen (10.10) — close in style, far in projected value.

**Value does not transfer across roles.** Substituting Haaland into four Arsenal defensive contexts collapses his rOBV to 1.37–2.35 against the original defenders' 3.63–8.78. The methodological point matters more than the football one: the model has **no positional labels**, so it cannot be penalising him for playing out of position. The decline comes purely from contextual demands.

## What ScoutGPT Changed

| | EventGPT | [[scoutgpt]] |
|---|---|---|
| Value handling | rOBV as a token | Auxiliary heads at outcome positions |
| Decoding | Unconstrained | [[constrained-decoding\|Validity-masked]] |
| Counterfactual | Re-score the **fixed** observed sequence | **Re-generate** the sequence |
| Event format | [[spadl]] | VERSA, with validity correction |

The substantive advance is the third row. EventGPT holds the event sequence fixed and re-evaluates rOBV under a substituted player, so it estimates how a player would *value* the same situations. ScoutGPT regenerates the sequence, so it estimates how a player would *change what happens*. The second is a stronger counterfactual and a harder problem.

## Limitations

- **Self-inconsistent baselines.** Saka's simulated rOBV (18.59) exceeds his ground truth (15.72), and the simulated figure is used as the comparison baseline — an acknowledged but uncorrected positive bias.
- **Training-window sensitivity.** An unusually low observed season (Núñez, 2023/24) propagates into the baseline against which substitutions are compared.
- Position-dependent aggregation (truncated mean for attackers) means values are not comparable across roles.
- On-ball events only; episode-level, not full-match.

## See Also

- [[scoutgpt]]
- [[on-ball-value]]
- [[player-embedding]]
- [[counterfactual-simulation]]
- [[large-event-model]]
- [[teacher-forcing]]
- [[eventgpt-player-impact-team-action-sequences|Source Summary]]
