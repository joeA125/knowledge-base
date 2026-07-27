---
title: "Kosuke Toda"
type: entity
tags: [person, researcher, university, sports-analytics, defensive-valuation, single-source]
sources: [raw/papers/football_defence_evaluation.md]
confidence: 0.7
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Kosuke Toda

Lead author of [[football-defence-evaluation-vdep|"Evaluation of soccer team defense based on prediction models of ball recovery and being attacked"]] (PLOS ONE, 2022), which introduced [[vdep]].

Graduate School of Human and Environmental Studies, [[kyoto-university]], at time of publication. Credited with conceptualisation, formal analysis, validation and visualisation — the analytical core of the paper — with [[keisuke-fujii]] as senior author from [[nagoya-university]].

## The Contribution

VDEP is the vault's first framework where **preventing** value is the modelling target rather than the negative half of an attacking model. Two ideas carry it:

- Replace the rare goal-conceded target with **frequent proxies** — ball recovery and effective attack. See [[rare-event-proxy-targets]].
- Add **off-ball state**: all 22 player positions and their distances from the ball, sorted by proximity, so the representation is permutation-invariant.

The empirical result that travels furthest beyond football is the demonstration that [[vaep]]'s conceding classifier scores **F1 = 0.000** on a 45-match dataset — identifying no true positives at all. That is a concrete measurement of a limitation the rest of the literature states qualitatively.

**Note:** vault knowledge of this person comes from a single paper. Nothing beyond this work is established.

## See Also

- [[vdep]] · [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[keisuke-kushiro]]
- [[kyoto-university]] · [[nagoya-university]]
- [[football-defence-evaluation-vdep|Source Summary]]
