---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, game-theory, proxy-target, counterfactual, optical-tracking-data]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project and JST PRESTO. **Senior author on six held sources** — more than twice anyone else in this vault.

## Six Primary Sources

| Year | Work | Lead author | Contribution |
|---|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] | [[kosuke-toda\|Toda]] | [[vdep]] — team defensive value from frequent proxies |
| 2022 | [[generalized-vdep-euro-location-analysis\|GVDEP]] | [[rikuhei-umemoto\|Umemoto]] | [[gvdep]] — score-scaled weights; partial-observation analysis |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]] — credit for [[space-creation\|space created]] for a teammate |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] | [[calvin-yeung\|Yeung]] | [[nmstpp]] and [[hpus]] |
| 2023 | [[team-defense-positioning-counterfactuals\|DRSO]] | Umemoto | [[drso]] — per-defender counterfactual positioning |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] | Yeung | [[game-theory\|Game-theoretic]] shot decisions; [[xsot\|xSOT]] |

## Two Signatures

### Change the target, not the model

Visible in four of six: **goals are too rare to model, so model something else on the causal path.**

| Work | Rare target abandoned | Proxy adopted |
|---|---|---|
| [[vdep]], [[gvdep]] | Goals conceded (186–227 positives per ~100k events) | Ball recovery, effective attack |
| [[hpus]] | Goals | Possession dynamics — **no goal data at any stage** |
| [[xsot]] | Goal | Shot on target — "the minimum requirement" |

Empirically justified from inside the group's own work: VDEP measures [[vaep|VAEP's]] conceding classifier at F1 = 0.000, GVDEP at 0.08–0.15. Having shown direct goal modelling fails at realistic data scale, proxy substitution stops being a convenience and becomes the correct response. See [[rare-event-proxy-targets]].

### Counterfactual on one named agent

The other half of the programme, and the one that produces **per-player** values:

| Work | Reference | Intervenes on |
|---|---|---|
| [[c-obso]] | Predicted "league average" trajectory | An attacker's movement |
| [[drso]] | Best reachable grid vertex | A defender's position |

Same machinery, opposite references — deviation from *normal* against deviation from *optimal*. Both build on [[obso|OBSO]] rather than event classification, which places this half of the group's work closer to [[william-spearman|Spearman's]] physical tradition than to VDEP's classifiers. See [[counterfactual-baseline]].

## Iteration Within the Group

**Each paper fixes a limitation the previous one named.** VDEP states three; GVDEP addresses all three. NMSTPP's stated gaps — no defensive actions, no player-level attribution — are addressed by VDEP and C-OBSO. VDEP's stated inability to individuate is addressed by DRSO.

That is rare in this literature, where papers more often propose alternatives than repair predecessors. It also means the group's own work is the closest thing the vault has to [[action-valuation-frameworks-compared|cross-framework comparison]] — GVDEP against VDEP on identical data — though same-group comparison is not an independent benchmark.

## Methodological Range

| Approach | Work |
|---|---|
| Supervised classification on engineered state | [[vdep]], [[gvdep]] |
| [[transformer]] [[point-process\|point process]] | [[nmstpp]] |
| [[graph-neural-network\|Graph]] [[variational-autoencoder\|VAE]] trajectory prediction | [[c-obso]] |
| [[theory-based-modelling\|Theory-based]] geometry + MLP + [[game-theory\|game theory]] | [[xsot]] |
| **Physical surface + counterfactual search, no ML** | **[[drso]]** |

No single architecture recurs. What recurs is **prediction or physics first, metric derived downstream** — none of the six defines value directly. That distinguishes the group from the Leuven line ([[vaep]]) and the Barcelona line ([[expected-value-possession-framework|EPV]]).

DRSO is the outlier in a telling way: it uses **no machine learning at all**, deliberately, on the argument that interpretability is what makes advice actionable for coaches.

## A Caution

Two of the group's papers ([[c-obso]], [[drso]]) set PPCF parameters $\sigma = 0.45$, $\lambda = 4.3$ citing [[beyond-expected-goals|Spearman (2018)]], which fits $s = 0.54$, $\lambda = 3.99$. A citation error propagating through the line — see [[obso]].

## Cited, Not Held

- **Teranishi, Fujii & Takeda (2020)**, IEEE GCCE pp. 124–125 — earliest of the trajectory line.
- **Fujii (2021)**, *Data-driven analysis for understanding team sports behaviors*, arXiv:2102.07545 — the group's own survey.
- **Nakahara et al. (2023)**, arXiv:2305.17886 — on/off-ball valuation via multi-agent deep RL.

## See Also

- [[vdep]] · [[gvdep]] · [[drso]] · [[c-obso]] · [[nmstpp]] · [[xsot]] · [[hpus]] · [[obso]]
- [[rare-event-proxy-targets]] · [[counterfactual-baseline]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]]
- [[game-theory]] · [[theory-based-modelling]] · [[interpretability]]
- [[calvin-yeung]] · [[kosuke-toda]] · [[rikuhei-umemoto]] · [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-kushiro]] · [[tony-sit]]
- [[nagoya-university]] · [[kyoto-university]] · [[william-spearman]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[optimal-decisions-shot-taking-situations|SPC Summary]]
