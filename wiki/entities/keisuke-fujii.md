---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, game-theory, proxy-target, optical-tracking-data]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Keisuke Fujii

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project and JST PRESTO. **Senior author on five held sources** — more than anyone else in this vault.

## Five Primary Sources

| Year | Work | Lead author | Contribution |
|---|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] | [[kosuke-toda\|Toda]] | [[vdep]] — team defensive value from frequent proxies |
| 2022 | [[generalized-vdep-euro-location-analysis\|GVDEP]] | [[rikuhei-umemoto\|Umemoto]] | [[gvdep]] — score-scaled weights; partial-observation analysis |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]] — credit for [[space-creation\|space created]] for a teammate |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] | [[calvin-yeung\|Yeung]] | [[nmstpp]] and [[hpus]] |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] | Yeung | [[game-theory\|Game-theoretic]] shot decisions; [[xsot\|xSOT]] |

## The Signature: Change the Target, Not the Model

Visible in four of five and stated explicitly in each: **goals are too rare to model, so model something else on the causal path.**

| Work | Rare target abandoned | Proxy adopted |
|---|---|---|
| [[vdep]], [[gvdep]] | Goals conceded (186–227 positives per ~100k events) | Ball recovery, effective attack |
| [[hpus]] | Goals | Possession dynamics — **no goal data at any stage** |
| [[xsot]] | Goal | Shot on target — "the minimum requirement" |

A research programme organised around one diagnosis, empirically justified from inside the group's own work: VDEP measures [[vaep|VAEP's]] conceding classifier at F1 = 0.000, and GVDEP at 0.08–0.15. Having shown that direct goal modelling fails at realistic data scale, substituting a proxy stops being a convenience and becomes the correct response. See [[rare-event-proxy-targets]].

## Iteration Within the Group

Unusual and worth noting: **each paper fixes a limitation the previous one named.** VDEP states three; GVDEP addresses all three. NMSTPP's stated gaps — no defensive actions, no player-level attribution — are addressed by VDEP and C-OBSO respectively, from the same group.

That is a rare pattern in this literature, where papers more often propose alternatives than repair predecessors. It also means the group's own work is the closest thing the vault has to [[action-valuation-frameworks-compared|cross-framework comparison]] — GVDEP against VDEP on identical data — though same-group comparison is not an independent benchmark.

## Two Routes to the Off-Ball Problem

**Outcome-proxy prediction** — [[vdep]] and [[gvdep]]. Predict frequent events on the causal path to conceding. Cheap, event-data-friendly, and **team-level only**.

**Trajectory and positional counterfactuals** — the Teranishi and Umemoto lines. Predict where players would have been, and value the difference. **Individuates naturally**, because the intervention is on one named player.

The second route is what makes individual credit possible, and the reason is general: [[counterfactual-baseline|intervention, not information, is what individuates]]. VDEP and C-OBSO use comparable data; only the one that intervenes produces a per-player number.

⚠️ **Note the distinction between two Umemoto papers.** The 2022 GVDEP work is held and is **team-level**. The 2023 counterfactual-positioning work — which would individuate defensive credit — is a different paper and is **not held**. See [[defensive-valuation]].

## Methodological Range

| Approach | Work |
|---|---|
| Supervised classification on engineered state | [[vdep]], [[gvdep]] |
| [[transformer]] [[point-process\|point process]] | [[nmstpp]] |
| [[graph-neural-network\|Graph]] [[variational-autoencoder\|VAE]] trajectory prediction | [[c-obso]] |
| [[theory-based-modelling\|Theory-based]] geometry + MLP + [[game-theory\|game theory]] | [[xsot]] |

No single architecture recurs. What recurs is **prediction first, metric derived downstream** — none of the five defines value directly. That distinguishes the group from the Leuven line ([[vaep]]) and the Barcelona line ([[expected-value-possession-framework|EPV]]).

## Cited, Not Held

- **Umemoto & Fujii (2023)**, StatsBomb Conference — individual defender evaluation via counterfactual positioning. **The vault's top acquisition target.**
- **Teranishi, Fujii & Takeda (2020)**, IEEE GCCE pp. 124–125 — earliest of the trajectory line.
- **Fujii (2021)**, *Data-driven analysis for understanding team sports behaviors*, J. Robot. Mechatron. 33(3) — the group's own survey, arXiv:2102.07545.

## See Also

- [[vdep]] · [[gvdep]] · [[c-obso]] · [[nmstpp]] · [[xsot]] · [[hpus]]
- [[rare-event-proxy-targets]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]]
- [[game-theory]] · [[theory-based-modelling]] · [[counterfactual-baseline]] · [[obso]]
- [[calvin-yeung]] · [[kosuke-toda]] · [[rikuhei-umemoto]] · [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-kushiro]] · [[tony-sit]]
- [[nagoya-university]] · [[kyoto-university]] · [[william-spearman]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
- [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[optimal-decisions-shot-taking-situations|SPC Summary]]
