---
title: "Tiago Mendes-Neves"
type: entity
tags: [person, researcher, university, sports-analytics, player-evaluation, time-series, generative-model]
sources: [raw/papers/football-performance-time-series.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-27
---

# Tiago Mendes-Neves

Researcher at the [[universidade-do-porto|Faculdade de Engenharia, Universidade do Porto]] and [[inesc-tec|LIAAD — INESC TEC]]. Lead author of the [[large-event-model|Large Event Model]] line of work with [[luis-meireles|Meireles]] and [[joao-mendes-moreira|Mendes-Moreira]], which frames football matches as token sequences and trains generative transformers on them — the closest thing in football analytics to a foundation-model programme.

## Primary Source in This Vault

[[football-performance-time-series|**"Valuing Players Over Time"**]] — the vault's first primary source by this author. It **predates** the LEM work and is methodologically quite different: a simplified [[vaep|VAEP]] variant using a [[random-forest|Random Forest]] regressor, with the contribution lying in [[player-rating-time-series|treating player ratings as time series]] rather than in the model.

Its ideas that carry beyond the paper:
- [[intent-vs-outcome-valuation|I-VAEP and O-VAEP]] — separating decision quality from execution quality by partitioning features on outcome-availability
- [[performance-volatility]] — consistency as downside deviation from a player's own long-term level, residualised against rating
- [[player-development-curve]] — age curves with an explicit [[selection-bias]] correction

## Other Papers Cited Across This Vault

- "Towards a foundation large events model for soccer" (*Machine Learning*, 2024)
- "Forecasting events in soccer matches through language" (2024)
- "A scalable approach for unified large events models in soccer" (ECML PKDD, 2026)

The LEM approach predicts a substantially more detailed action set than contemporaries — 33 action types against [[nmstpp]]'s 5 and [[sig-model]]'s 7 — alongside gridded location and elapsed time.

## Trajectory

The two strands show a clear progression in ambition. The time-series paper takes an existing valuation model as given and asks what better *aggregation* can extract from it. The LEM papers discard the hand-built pipeline entirely in favour of learning the event distribution directly.

What persists across both is a conviction that football event streams should be treated as **sequential data with structure worth modelling**, rather than as a bag of actions to be summed — the time-series paper makes that argument at the level of match-by-match ratings, the LEM papers at the level of individual events.

> **Provenance note.** Knowledge of the LEM papers still comes only from citations in [[scoutgpt-counterfactual-player-valuation|Hong et al. (2026)]] and [[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]], not from the primary sources. The time-series paper is held directly.

## See Also

- [[luis-meireles]] · [[joao-mendes-moreira]]
- [[universidade-do-porto]] · [[inesc-tec]] · [[fc-porto]]
- [[large-event-model]] · [[scoutgpt]] · [[seq2event]] · [[nmstpp]]
- [[player-rating-time-series]] · [[intent-vs-outcome-valuation]]
- [[football-performance-time-series|Source Summary]]
