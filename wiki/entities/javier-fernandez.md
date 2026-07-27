---
title: "Javier Fernández"
type: entity
tags: [person, researcher, sports-analytics, action-valuation, optical-tracking-data]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/football-performance-time-series.md]
confidence: 0.65
provenance:
  extracted: 35%
  inferred: 50%
  ambiguous: 15%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Javier Fernández

Lead author of the deep-learning [[expected-possession-value|expected possession value]] framework for soccer, with [[luke-bornn]] and [[daniel-cervone]] — the work that carried the basketball [[martingale-epv|EPV]] programme across to football.

Associated with FC Barcelona's analytics department during this work.

## Why the Page Exists

Fernández has been referenced on [[expected-possession-value]] since that page was written, without a page of his own, while his two co-authors both have one. The gap is worth closing because his contribution is the specific bridge between the vault's basketball and football valuation literatures — and because the "soccer EPV" terminology confusion that page documents is largely downstream of this work.

## The Contribution

[[martingale-epv|Cervone et al.'s basketball model]] depends on 25 Hz [[optical-tracking-data|optical tracking]] and enormous computation. Fernández, Bornn & Cervone reconstruct the same *conceptual* object for soccer — a continuously-updating estimate of a possession's worth — using deep learning over tracking data, decomposing possession value into components for passing, driving, and shooting.

This matters for how "EPV" is used in soccer. The label had already been claimed by the [[expected-threat|xT]]-style zonal Markov models, which are far simpler and satisfy none of the basketball model's stated criteria. Fernández's framework is a genuine Cervone-style construction wearing the same name, which is precisely why the term now means two incompatible things in the literature. See the terminology warning on [[expected-possession-value]].

## Dating

The vault previously dated this framework to 2019. The fuller journal version is **2021** — *Machine Learning* 110(6), 1389–1427, "A framework for the fine-grained evaluation of the instantaneous expected value of soccer possessions" — with an earlier MIT Sloan conference version in 2019. Both dates appear in the literature; the [[epv-control-duel-skills-football|Shelopugin paper]] cites the 2019 conference version.

**Note:** vault knowledge of this person comes from citations within other papers rather than from the primary source, which is not held in `raw/`. Acquiring it would be worthwhile — it is the most-cited football EPV reference the vault lacks.

## See Also

- [[expected-possession-value]] · [[martingale-epv]]
- [[luke-bornn]] · [[daniel-cervone]] · [[kirk-goldsberry]]
- [[optical-tracking-data]] · [[action-valuation]]
- [[action-valuation-frameworks-compared]]
