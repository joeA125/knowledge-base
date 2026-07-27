---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, optical-tracking-data, counterfactual]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Keisuke Fujii

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project (Fukuoka) and JST PRESTO. Senior author on both of the vault's Japanese football-analytics sources, and lead of the most sustained **defensive and off-ball valuation** programme in this literature.

## Two Primary Sources Here

| Year | Work | Contribution |
|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] (Toda, Teranishi, Kushiro & Fujii) | [[vdep]] — team defensive value from frequent-proxy prediction |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] (Yeung, Sit & Fujii) | [[nmstpp]] and [[hpus]] |

## The Methodological Signature

Fujii's group comes at football through **prediction rather than valuation**. Neither held source defines value directly; both fit a predictive model and derive a metric downstream:

- VDEP predicts ball recovery and being attacked, then combines the two probabilities.
- NMSTPP predicts the next event's time, zone and action, and [[hpus]] falls out of the forecasts.

This distinguishes the group from the Leuven line ([[vaep]], value as a probability difference) and the Barcelona line ([[expected-value-possession-framework|EPV]], value as a conditional expectation, decomposed). The practical payoff is that both metrics sidestep goal sparsity — HPUS uses **no goal data at any stage**, and VDEP abandons the goal target explicitly. See [[rare-event-proxy-targets]].

## The Defensive and Off-Ball Programme

> **Provenance warning.** None of the four works below is held in `raw/`. Bibliographic details were verified against citation lists and the author's own overview article; **method descriptions and capability claims are as stated by the authors and are unverified here.**

| Year | Work | Contribution |
|---|---|---|
| 2020 | Teranishi, Fujii & Takeda — *Trajectory prediction with imitation learning reflecting defensive evaluation in team sports*, IEEE GCCE, pp. 124–125 | Earliest of the line; trajectory-based defensive evaluation. A two-page abstract. |
| 2022 | Umemoto, **Tsutsui** & Fujii — *Location analysis of players in UEFA EURO 2020 and 2022 using generalized valuation of defense by estimating probabilities*, arXiv:2212.00021 | **GVDEP** — generalises [[vdep]] to player-location level |
| 2022/23 | Teranishi, **Tsutsui**, Takeda & Fujii — *Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction*, MLSA / Springer, pp. 53–73 | Credits **movement sacrificed for a teammate** — space creation |
| 2023 | **Umemoto & Fujii** — *Evaluation of team defense positioning by computing counterfactuals using StatsBomb 360 data*, StatsBomb Conference | **Individual defender evaluation** via counterfactual positioning |

**Correction, 2026-07-27.** This page previously listed these works under paraphrased descriptions rather than titles, omitted Kazushi Tsutsui from two author lists, and missed the 2023 counterfactual positioning paper entirely. The paraphrases made the papers effectively unfindable by search — a reminder that citation-derived entries should record titles verbatim.

### Two routes, converging

The programme runs on two distinct mechanisms aimed at the same target — valuing players who never touch the ball:

**Outcome-proxy prediction** — VDEP and GVDEP. Predict frequent events on the causal path to conceding. Cheap, event-data-friendly, and in VDEP's case team-level only.

**Trajectory and positional counterfactuals** — the Teranishi and Umemoto lines. Predict where players would have been, and value the difference. Individuates naturally, because the intervention is on one named player.

Fujii describes combining the two as a framework covering the movements of almost all players. He also reports the binding constraint: the trajectory method needed **enormous computation to evaluate all 22 players**, requiring a separate trajectory prediction per player evaluated — the same cost wall that put [[martingale-epv|Cervone et al.]] on 461 processors.

### The OBSO substrate

Both counterfactual lines build on **OBSO** (off-ball scoring opportunities, Spearman 2018) rather than on event classification. That places the group's off-ball work closer in spirit to [[probability-surface|Fernández et al.'s]] value surfaces than to VDEP's classifiers, despite the shared authorship — and makes Spearman (2018) a notable gap in `raw/`.

## Acquisition Priority

The three arXiv-available works — GVDEP (2212.00021), the multi-agent RL action valuation paper (2305.17886), and Fujii's survey *Data-Driven Analysis for Understanding Team Sports Behaviors* (2102.07545, J. Robot. Mechatron. 33(3)) — are fetchable now. The survey is reference [3] in the VDEP paper and maps the whole programme.

The 2023 counterfactual positioning work is the highest-value target for closing the vault's individual-defender gap, but was presented at a conference rather than published as a preprint, so availability is uncertain.

## See Also

- [[vdep]] · [[defensive-valuation]] · [[off-ball-value]] · [[rare-event-proxy-targets]]
- [[nmstpp]] · [[hpus]] · [[action-valuation]] · [[counterfactual-simulation]] · [[probability-surface]]
- [[kosuke-toda]] · [[masakiyo-teranishi]] · [[keisuke-kushiro]] · [[calvin-yeung]] · [[tony-sit]]
- [[nagoya-university]] · [[kyoto-university]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[transformer-point-process-football-event-modelling|NMSTPP Summary]]
