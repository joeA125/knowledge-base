---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, game-theory, proxy-target, optical-tracking-data]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Keisuke Fujii

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project (Osaka/Fukuoka) and JST PRESTO. **Senior author on four held sources** — more than anyone else in this vault.

## Four Primary Sources

| Year | Work | Lead author | Contribution |
|---|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] | [[kosuke-toda\|Toda]] | [[vdep]] — team defensive value from frequent proxies |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]] — credit for [[space-creation\|space created]] for a teammate |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] | [[calvin-yeung\|Yeung]] | [[nmstpp]] and [[hpus]] |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] | Yeung | [[game-theory\|Game-theoretic]] shot decisions; [[xsot\|xSOT]] |

## The Signature: Change the Target, Not the Model

The group's defining move, visible in three of the four and stated explicitly in each: **goals are too rare to model, so model something else on the causal path.**

| Work | Rare target abandoned | Proxy adopted |
|---|---|---|
| [[vdep]] | Goals conceded (227 in 97,335 events) | Ball recovery, effective attack |
| [[hpus]] | Goals | Possession dynamics — **no goal data at any stage** |
| [[xsot]] | Goal | Shot on target — "the minimum requirement" |

That is not three applications of a trick; it is a research programme organised around one diagnosis. See [[rare-event-proxy-targets]].

The diagnosis is empirically justified from inside the group's own work: VDEP measures [[vaep|VAEP's]] conceding classifier at **F1 = 0.000** — a goal-targeted model identifying no true positives at all. Having demonstrated that direct goal modelling fails at realistic data scale, substituting a proxy stops being a convenience and becomes the correct response.

## Two Routes to the Off-Ball Problem

The group attacks the blind spot shared by [[vaep]], [[expected-threat|xT]] and [[nmstpp]] itself from two directions:

**Outcome-proxy prediction** — [[vdep]] and GVDEP. Predict frequent events on the causal path to conceding. Cheap, event-data-friendly, and **team-level only**.

**Trajectory and positional counterfactuals** — the Teranishi and Umemoto lines. Predict where players would have been, and value the difference. **Individuates naturally**, because the intervention is on one named player.

The second route is what makes individual credit possible, and the reason is general: [[counterfactual-baseline|intervention, not information, is what individuates]]. VDEP and C-OBSO use comparable data; only the one that intervenes on a named player produces a per-player number.

Both counterfactual lines build on [[obso|OBSO]] rather than event classification, placing that work closer to [[william-spearman|Spearman's]] physical tradition than to VDEP's classifiers despite the shared authorship.

## Methodological Range

Unusually broad for one group, and the range is itself informative about the problem:

| Approach | Work |
|---|---|
| Supervised classification on engineered state | [[vdep]] |
| [[transformer]] [[point-process\|point process]] | [[nmstpp]] |
| [[graph-neural-network\|Graph]] [[variational-autoencoder\|VAE]] trajectory prediction | [[c-obso]] |
| [[theory-based-modelling\|Theory-based]] geometry + MLP + [[game-theory\|game theory]] | [[xsot]] |

No single architecture recurs. What recurs is **prediction first, metric derived downstream** — none of the four defines value directly. That distinguishes the group from the Leuven line ([[vaep]], value as a probability difference) and the Barcelona line ([[expected-value-possession-framework|EPV]], value as a decomposed conditional expectation).

## The Defensive Programme

> **Provenance.** [[vdep]] and [[c-obso]] are held. The two below are **cited only** — bibliographic details verified, capability claims unverified here.

- **Teranishi, Fujii & Takeda (2020)**, IEEE GCCE pp. 124–125 — earliest of the trajectory line.
- **Umemoto, Tsutsui & Fujii (2022)**, arXiv:2212.00021 — **GVDEP**, generalising VDEP to player-location level.
- **Umemoto & Fujii (2023)**, StatsBomb Conference — **individual defender evaluation** via counterfactual positioning. Remains the vault's top acquisition target, since individual defensive credit is the last major uncovered capability.

Fujii reports the binding constraint on the trajectory route: **enormous computation to evaluate all 22 players**, one prediction per player. [[c-obso]] confirms it — only three of 22 are predicted.

## See Also

- [[vdep]] · [[c-obso]] · [[nmstpp]] · [[xsot]] · [[hpus]]
- [[rare-event-proxy-targets]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]]
- [[game-theory]] · [[theory-based-modelling]] · [[counterfactual-baseline]] · [[obso]]
- [[calvin-yeung]] · [[kosuke-toda]] · [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-kushiro]] · [[tony-sit]]
- [[nagoya-university]] · [[kyoto-university]] · [[william-spearman]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
- [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[optimal-decisions-shot-taking-situations|SPC Summary]]
