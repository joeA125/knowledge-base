---
title: "Javier Fernández"
type: entity
tags: [person, researcher, sports-analytics, action-valuation, optical-tracking-data, deep-learning, probability-surface]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/football-performance-time-series.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Javier Fernández

Lead author of the deep-learning [[expected-possession-value|expected possession value]] framework for soccer, with [[luke-bornn]] and [[daniel-cervone]] — the work that carried the basketball [[martingale-epv|EPV]] programme across to football.

Led analytics at [[fc-barcelona]] during this work.

## Line of Work

Three connected papers, the first two with Bornn:

| Year | Work | Contribution |
|---|---|---|
| 2018 | Wide Open Spaces (MIT Sloan) | [[pitch-control]] and pitch influence via player reachability surfaces |
| 2020 | SoccerMap (arXiv) | [[soccermap]] — the fully convolutional surface architecture |
| 2020/21 | [[expected-value-possession-framework\|EPV framework]] | Combines both into a decomposed, tracking-based EPV |

The EPV framework is the capstone: pitch control enters as an input feature, SoccerMap serves as the feature extractor for all three pass components, and the [[structured-model-decomposition|decomposition]] binds them into a single calibrated estimate.

## The Contribution

[[martingale-epv|Cervone et al.'s basketball model]] depends on 25 Hz [[optical-tracking-data|optical tracking]] and enormous computation. Fernández, Bornn & Cervone reconstruct the same *conceptual* object for soccer — a continuously-updating estimate of a possession's worth — but arrive at it by a very different route: nine separately-trained neural components rather than one Bayesian process model, and real-time inference rather than 461 processors.

Two things distinguish the work within the vault.

**It values off-ball positioning.** Because pass value is estimated at every pitch location, the worth of a player *standing* somewhere falls out for free. This is the vault's only substantive treatment of what every other framework lists as a shared limitation. See [[off-ball-value]].

**It argues interpretability is compatible with model richness.** The vault's valuation comparison shows a consistent trade-off — richer state costs legibility. Fernández's answer is to decompose along axes coaches already use rather than to simplify. Whether that fully succeeds is arguable, but it is the only serious attempt here to escape the trade-off rather than pick a side.

## On the Terminology Confusion

The "EPV" label had already been claimed in soccer by [[expected-threat|xT]]-style zonal Markov models, which are far simpler and satisfy none of the basketball model's stated criteria. Fernández's framework is a genuine Cervone-style construction wearing the same name — which is a substantial part of why the term now means several incompatible things. See the terminology warning on [[expected-possession-value]].

## Dating

The vault previously carried an unresolved note about whether to date this work 2019 or 2021. With the primary source held, the sequence is:

- **2019** — MIT Sloan conference version, "Decomposing the immeasurable sport"
- **2020** — arXiv:2011.09426, 18 November (the version in `raw/`)
- **2021** — *Machine Learning* 110(6), 1389–1427

Same line of work at increasing length. [[epv-control-duel-skills-football|Shelopugin]] cites the 2019 version; [[football-performance-time-series|Mendes-Neves et al.]] cite the 2021 one.

## See Also

- [[expected-value-possession-framework|Source Summary]]
- [[expected-possession-value]] · [[martingale-epv]] · [[action-valuation]]
- [[soccermap]] · [[pitch-control]] · [[probability-surface]] · [[off-ball-value]]
- [[structured-model-decomposition]] · [[policy-modelling]]
- [[luke-bornn]] · [[daniel-cervone]] · [[fc-barcelona]]
- [[action-valuation-frameworks-compared]]
