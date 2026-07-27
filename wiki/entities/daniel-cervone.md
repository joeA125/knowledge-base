---
title: "Daniel Cervone"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, stochastic-process, optical-tracking-data]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Daniel Cervone

Moore-Sloan Data Science Fellow at the Center for Data Science, New York University (at time of publication). Lead author of [[multiresolution-stochastic-process-nba-possessions|A Multiresolution Stochastic Process Model for Predicting Basketball Possession Outcomes]], which introduced [[martingale-epv]] and the [[multiresolution-modelling]] framework for estimating it from [[optical-tracking-data]].

The work is cited as related work in [[evaluating-football-player-actions|Decroos et al. (2019)]], the soccer [[vaep]] paper, as one of the foundational Markov-process approaches to valuing player actions — and by [[epv-control-duel-skills-football|Shelopugin]] as the origin of the EPV idea itself.

## Two Models, Opposite Design Philosophies

Cervone is an author on both the basketball construction and its soccer successor, [[expected-value-possession-framework|Fernández, Bornn & Cervone (2020)]] — and the two differ on nearly every methodological choice:

| | 2016 basketball | 2020 soccer |
|---|---|---|
| Estimation | One Bayesian process model | Nine supervised components |
| [[martingale]] guarantee | **Yes** — the defining feature | No |
| Interpretability from | Stochastic consistency | [[structured-model-decomposition\|Decomposition]] |
| Cost | 461 processors | Real-time |

The 2016 paper argues explicitly that regressing game-state features onto outcomes cannot produce an interpretable value curve. The 2020 framework does structurally that. Cervone's presence on both makes this a **considered revision of what interpretability requires** rather than a lapse — from a mathematical property of the estimator to a structural property of the model. See [[martingale-epv]] for the full comparison.

Reading the two together is the best available illustration in this vault that the martingale property is a cost as well as a virtue: it is what forced 461 processors, and abandoning it is what made a usable real-time coaching tool possible.

## Note on Dating

Earlier vault notes dated the soccer framework to the MIT Sloan conference version (2019). With the arXiv preprint now held, the sequence is 2019 conference → 2020 arXiv → 2021 *Machine Learning* journal. All three are the same work.

Maintains a public demo repository with sample tracking data and R code for reproducing EPV calculations.

## See Also

- [[martingale-epv]] · [[expected-possession-value]] · [[multiresolution-modelling]]
- [[expected-value-possession-framework|Soccer EPV Framework Summary]] · [[structured-model-decomposition]]
- [[luke-bornn]] · [[javier-fernandez]] · [[alex-damour]] · [[kirk-goldsberry]]
