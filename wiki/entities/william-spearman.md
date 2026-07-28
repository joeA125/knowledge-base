---
title: "William Spearman"
type: entity
tags: [person, researcher, practitioner, sports-analytics, off-ball, pitch-control]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.7
provenance:
  extracted: 40%
  inferred: 45%
  ambiguous: 15%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# William Spearman

Creator of [[obso|OBSO]] (off-ball scoring opportunity) and the physics-based **pitch control** model underlying it. Associated with Liverpool FC's research department.

## Why He Matters Here

Spearman is the most-cited author in this vault who has no primary source held. Two independent lines depend on his work:

- The **[[keisuke-fujii|Fujii group]]** builds both its counterfactual methods on OBSO — [[c-obso]] for attacking space creation, and Umemoto & Fujii (2023) for defensive positioning.
- **[[expected-value-possession-framework|Fernández, Bornn & Cervone]]** cite "Beyond Expected Goals" as the closest prior work on off-ball valuation, and their [[pitch-control]] construction is an alternative to his.

## Two Contributions

**Physics-based pass and control modelling** (Spearman et al., 2017) — treats a player's ability to control the ball as a **Poisson point process**, with control probability accumulating the longer a player is near the ball uncontested, and shared competitively across all players. This is the PPCF term inside OBSO. See [[obso]].

**[[obso|OBSO]]** (2018, MIT Sloan) — factorises the value of an off-ball position into control × transition × score, each a separately modelled surface over the pitch. Simple, interpretable, and rule-based rather than learned.

## A Note on Two Pitch-Control Traditions

The vault holds two constructions of the same object, and they differ in kind:

| | Spearman | [[pitch-control\|Fernández & Bornn]] |
|---|---|---|
| Mechanism | Arrival-time contest, Poisson control | Gaussian influence density |
| Grounding | Physical — time to reach, time to control | Statistical — reachability as a distribution |
| Competition | Explicit, via shared probability mass | Difference of summed influences |

Spearman's is more principled about *why* a player controls a location; Fernández & Bornn's is cheaper and smoother. **No source in this vault compares them**, and both are used as inputs to downstream value models — so a difference between the two propagates silently into everything built on top.

## Acquisition Priority

"Beyond Expected Goals" (12th MIT Sloan Sports Analytics Conference, 2018) is now the **highest-priority missing source** in the vault's football coverage. It is a dependency of at least four held pages and the substrate for two separate research lines, and everything the vault knows about it is second-hand.

"Physics-based modeling of pass probabilities in soccer" (11th MIT Sloan, 2017), with Basye, Dick, Hotovy and Pop, supplies the PPCF parameters ($s = 0.45$, $\lambda = 4.3$) used unchanged by Teranishi et al.

**Note:** vault knowledge of this person comes entirely from citations and descriptions within other papers. Affiliation is inferred from the wider literature rather than stated in any held source.

## See Also

- [[obso]] · [[pitch-control]] · [[off-ball-value]] · [[c-obso]]
- [[expected-goals]] · [[probability-surface]] · [[keisuke-fujii]] · [[javier-fernandez]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
