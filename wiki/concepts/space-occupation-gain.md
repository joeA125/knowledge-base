---
title: "Space Occupation Gain and Space Generation Gain"
type: concept
tags: [space-creation, off-ball, sports-analytics, player-evaluation, pitch-control, optical-tracking-data, tactical-analysis, evaluation]
sources: [raw/papers/wide_open_spaces_creation_football.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  generated: 4%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Space Occupation Gain and Space Generation Gain

[[wide-open-spaces-space-creation|Fernández & Bornn's (2018)]] paired metrics for **space created for oneself** and **space created for a teammate**.

The vault referenced these for months as "the other route into [[space-creation]]" before holding the source. They are the earlier of the two held approaches — [[c-obso|C-OBSO]] arrives four years later by a different mechanism.

## The Quantity Being Tracked

Space is worth having only where it is valuable, so quality of owned space multiplies control by value:

$$Q_i(t) = PC_i(t)\,V(t)$$

with $PC$ from the Gaussian [[pitch-control]] model and $V$ from the [[pitch-value-model|ball-relative pitch value model]]. Both are needed: control of the corner flag is control of nothing.

## Space Occupation Gain

Mean change in $Q$ over a window $[t+1, t+w+1]$, with $w = 3$ seconds:

$$G_i(t) = \frac{1}{w}\sum_{t'=t+1}^{t+w+1} Q_i(t')$$

$$SOG_i(t) = G_i(t) \ \text{if} \ G_i(t) \ge \epsilon, \ \text{else } 0$$

The threshold $\epsilon$ exists because **football is a continuous process of winning and losing space**. Without it the metric would count ordinary drift — a defender following the ball away from a player technically "gains" that player space. $\epsilon$ asks for a gain large enough to read as an occupational advantage.

The symmetric quantity, Space Occupation Loss, is the negative case.

### Active and passive

Occupation above jogging pace (**1.5 m/s**) is *active*; below is *passive*.

The distinction earns its place through one finding. Across Barcelona's outfield players, active occupation dominates — full-backs Digne and Sergi Roberto sit near 80% active. **Messi is the exception at 66.7% passive**, and the paper reads it directly:

> *"that walking behaviour is not a detachment from the match but a conscious action to move through empty spaces of value and claim the control of valuable space."*

A composite SOG would show Messi third in the squad and say nothing about how. The split says he does it by walking into value rather than running at it — and 71% of his gain happens in front of the ball. Another instance of the vault's recurring [[capability-profiling|decomposition-beats-composite]] pattern.

## Space Generation Gain

Generation is a **logical predicate on dragging**, not a model. For teammates $i, i'$ and opponent $j$ over window $w$:

$$SG_{i,i'}(t) = \exists j \ (d_{i',j}(t) \le \delta) \wedge (d_{i,j}(t+w) \le \delta) \wedge (d_{i',j}(t+w) > \delta) \wedge (d_{i,j}(t+w) - d_{i,j}(t) < \alpha)$$

In words: an opponent starts near teammate $i'$, ends near generator $i$, leaves $i'$, and travelled far enough for the attraction to be real. $\delta = 5$ m (the average marking distance), $\alpha = 3$ m.

$SGG$ is then the freed space where the gain clears $\epsilon$.

**The credit is attributed by co-occurrence, not causation.** A defender who leaves for entirely unrelated reasons — tracking the ball, following a coaching instruction — still credits the "generator". Nothing in the predicate distinguishes being dragged from happening to move.

That is the sharpest structural difference from [[c-obso]], which asks a **counterfactual** — what would have happened had this player moved as predicted — rather than pattern-matching a spatial configuration.

## What the Metrics Separate

| Player | Occupation | Generation |
|---|---|---|
| Iniesta, Busquets, Messi | **41% of team SOG between them** | Mid-table |
| Neymar, Suárez | Lower, attributed to close marking | **Highest generation** |
| Digne, S. Roberto (full-backs) | Moderate | **Near zero** |

**Occupation and generation are different skills**, and a player strong at one may be weak at the other. Suárez concentrates his generation inside the box; full-backs generate almost nothing because moving toward a touchline drags nobody useful.

The generator–receiver matrix shows structure beyond individual totals — a reciprocal Suárez↔Messi pair, and Busquets receiving from nearly everyone, which the authors read as third-man-pass behaviour. That is a **relational** output no per-player rating expresses, and the only other framework here producing one is [[c-obso]].

## Compared with C-OBSO

| | **SOG / SGG** | [[c-obso]] |
|---|---|---|
| Year | 2018 | 2022/23 |
| Credits generation via | **Spatial predicate on dragging** | **Counterfactual against predicted movement** |
| Value substrate | Pitch control × ball-relative value | [[obso\|OBSO]] |
| Causal claim | Co-occurrence | Deviation from a reference |
| Needs a trained model | Only for pitch value | Yes — GVRNN trajectory predictor |
| Fails when | Defender moves for unrelated reasons | The predictor is wrong |
| External validation | None — one match | Salary correlation 0.45 |

Neither dominates. The predicate is cheap, transparent and attributes by coincidence; the counterfactual is principled and inherits its predictor's errors — and is identically zero under a perfect predictor. See [[counterfactual-baseline]].

## Limitations

- **One match.** All findings rest on Barcelona–Villarreal, January 2017, 845 situations.
- **Four asserted parameters** — $w$, $\delta$, $\alpha$, $\epsilon$ — none swept. See [[free-parameters-load-bearing]].
- **Unoccupied generated space is explicitly excluded.** Dragging a defender out of a zone nobody enters counts for nothing, though the authors name it as a case they chose not to model.
- **No ground truth**, acknowledged by the authors; validation is expert video review.
- **No [[split-half-reliability|reliability]] figure**, and a single match cannot supply one.
- **Marking intensity confounds occupation.** Neymar and Suárez score low on SOG, which the paper attributes to close marking rather than poor movement — so the metric partly measures how opponents treat a player.

## See Also

- [[space-creation]] · [[c-obso]] · [[pitch-value-model]] · [[pitch-control]] · [[off-ball-value]]
- [[counterfactual-baseline]] · [[capability-profiling]] · [[free-parameters-load-bearing]] · [[obso]]
- [[javier-fernandez]] · [[luke-bornn]] · [[fc-barcelona]]
- [[wide-open-spaces-space-creation|Source Summary]] · [[action-valuation-frameworks-compared]]
