---
title: "Masakiyo Teranishi"
type: entity
tags: [person, researcher, university, sports-analytics, off-ball, space-creation, trajectory-prediction, counterfactual]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/football_defence_evaluation.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Masakiyo Teranishi

Researcher at the Graduate School of Informatics, [[nagoya-university]]. **Lead author of [[creating-scoring-opportunities-trajectory-prediction|the C-OBSO paper]]**, and the author most consistently associated with the [[keisuke-fujii|Fujii group's]] trajectory-modelling line.

*(Previously a citation-only stub recorded during the VDEP ingest. Rewritten on acquisition of the primary source.)*

## Line of Work

| Year | Work | Role | Held? |
|---|---|---|---|
| 2020 | *Trajectory prediction with imitation learning reflecting defensive evaluation in team sports*, IEEE GCCE, pp. 124–125 | First author | No |
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] (Toda, Teranishi, Kushiro & Fujii) | Data curation, software | **Yes** |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] (Teranishi, Tsutsui, Takeda & Fujii), MLSA / Springer | **First author** | **Yes** |

The 2020 abstract and C-OBSO are two ends of one idea: predict how players *would* have moved, then read meaning from the difference. The 2020 work applied it to defensive evaluation; C-OBSO applies it to attacking space creation and adds the [[obso|OBSO]] substrate and a graph-based predictor.

## The Contribution

C-OBSO is the vault's only framework that assigns value **relationally** — from the player who moves to the player who benefits. An off-ball player is credited with the improvement in *someone else's* scoring chance attributable to his deviating from predicted movement. See [[counterfactual-baseline]].

Two portable ideas:

**Prediction as a reference rather than a forecast.** Train a trajectory model on opponents to generate "league average" movement, then treat the prediction as the counterfactual. The objective is a well-calibrated notion of *normal*, not minimal error — and the two goals are in genuine tension, since C-OBSO is identically zero under perfect prediction.

**A defender-aware score model.** Replacing OBSO's distance-only scoring term with per-degree shot value discounted by a Gaussian shot-blocking distribution. Modest in RMSE (0.309 vs 0.324) but qualitatively sharper, separating shots from equal distance by defender congestion.

His role on VDEP was data curation and software rather than method, so the two held sources show him in different capacities.

## See Also

- [[c-obso]] · [[obso]] · [[trajectory-prediction]] · [[counterfactual-baseline]]
- [[off-ball-value]] · [[defensive-valuation]] · [[vdep]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[kosuke-toda]]
- [[nagoya-university]] · [[william-spearman]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
