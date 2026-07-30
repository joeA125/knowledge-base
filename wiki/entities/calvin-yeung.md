---
title: "Calvin C. K. Yeung"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, game-theory, event-prediction]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 22%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Calvin C. K. Yeung

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Lead author of two held sources, both with [[keisuke-fujii]] as senior author.

| Year | Work | Contribution |
|---|---|---|
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] (Yeung, Sit & Fujii) | [[nmstpp]] and the [[hpus]] metric |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] (Yeung & Fujii) | [[game-theory\|Game-theoretic]] shot analysis; [[xsot\|xSOT and xOSOT]] |

## The Through-Line

Both papers attack the same underlying obstacle — **goals are too rare to model directly** — and both solve it by changing the target rather than the model.

NMSTPP forecasts event time, zone and action, from which [[hpus]] is derived using **no goal data at any stage**. The SPC framework substitutes *shot on target* for *goal* as the game-theoretic payoff, on the grounds that it is the minimum requirement of a good shot. Both are instances of [[rare-event-proxy-targets]], and together with [[vdep]] they make proxy substitution a signature of the [[keisuke-fujii|Fujii group]] rather than a one-off.

The 2024 paper is the more unusual of the two, and not only for football. It is the vault's **first framework to compute an optimal policy** rather than evaluate an observed one — achieved by restricting to a two-action game where payoffs for unobserved strategy profiles are estimable rather than extrapolated. See [[policy-modelling]] for why that restriction is what makes it work.

## Also Cited, Not Held

- Yeung, Bunker & Fujii (2023), *A framework of interpretable match results prediction in football with FIFA ratings and team formation*, PLOS ONE — proposed as the route to player-specific xSOT.
- Yeung & Bunker (2023), *An events and 360 data-driven approach for extracting team tactics*, StatsBomb Conference.

Supported by JST SPRING (JPMJSP2125) and the Interdisciplinary Frontier Next-Generation Researcher Program, Tokai Higher Education and Research System. Maintains public implementations of both models.

## See Also

- [[nmstpp]] · [[hpus]] · [[event-prediction]] · [[point-process]]
- [[game-theory]] · [[xsot]] · [[theory-based-modelling]] · [[policy-modelling]]
- [[rare-event-proxy-targets]] · [[keisuke-fujii]] · [[tony-sit]] · [[nagoya-university]]
- [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[optimal-decisions-shot-taking-situations|SPC Summary]]
