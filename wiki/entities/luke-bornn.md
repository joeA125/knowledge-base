---
title: "Luke Bornn"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, spatiotemporal, pitch-control]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Luke Bornn

Assistant Professor in the Department of Statistics and Actuarial Science at Simon Fraser University (at time of publication). Co-author of [[multiresolution-stochastic-process-nba-possessions|A Multiresolution Stochastic Process Model for Predicting Basketball Possession Outcomes]] (2016), which introduced [[martingale-epv]].

A prolific contributor to spatial and spatio-temporal sports analytics, also co-authoring the defensive-structure work (Franks et al., 2015) whose defender-assignment model the EPV microtransition model builds on, and the factorised point process work (Miller et al., 2013) that informs its [[non-negative-matrix-factorization|NMF]]-based spatial bases.

## The Bridge Between Two Sports

Bornn is the connective figure in this vault's possession-value literature, appearing on both sides of the basketball-to-soccer transfer:

| Year | Work | With | Contribution |
|---|---|---|---|
| 2016 | Multiresolution EPV | Cervone, D'Amour, Goldsberry | [[martingale-epv]] |
| 2018 | Wide Open Spaces | [[javier-fernandez\|Fernández]] | [[pitch-control]] and pitch influence |
| 2020 | SoccerMap | Fernández | [[soccermap]] architecture |
| 2020/21 | [[expected-value-possession-framework\|Soccer EPV framework]] | Fernández, Cervone | Decomposed tracking-based EPV |

The pattern is consistent across all four: **turn positional tracking data into a spatial field, then reason over the field rather than over events.** In basketball that yields spatial bases and defender-assignment models; in soccer it yields pitch-control surfaces and eventually full-pitch [[probability-surface|value surfaces]].

Notably, the soccer line abandons the martingale guarantee that defined the basketball work — a trade discussed on [[martingale-epv]]. Bornn's presence on both makes that a considered change of approach rather than an oversight by newcomers.

The 2018 *Wide Open Spaces* work is also the vault's other route into [[space-creation]], measuring space as **area opened** where [[c-obso]] measures it as value transferred to a teammate.

Funded at the time by DARPA, the US Army Research Office, Amazon AWS, and NSERC.

## See Also

- [[martingale-epv]] · [[expected-possession-value]] · [[pitch-control]] · [[stochastic-process]]
- [[soccermap]] · [[probability-surface]] · [[off-ball-value]] · [[space-creation]]
- [[gaussian-process]] · [[optical-tracking-data]] · [[multiresolution-modelling]]
- [[daniel-cervone]] · [[javier-fernandez]] · [[kirk-goldsberry]] · [[william-spearman]]
- [[expected-value-possession-framework|Soccer EPV Framework Summary]]
