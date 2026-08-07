---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, game-theory, proxy-target, counterfactual, reinforcement-learning, multi-agent, optical-tracking-data]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.9
provenance:
  extracted: 74%
  inferred: 21%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-08-07
---

# Keisuke Fujii

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project and JST PRESTO. **Senior author on seven held sources** — more than twice anyone else in this vault.

## Seven Primary Sources

| Year | Work | Lead author | Contribution |
|---|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] | [[kosuke-toda\|Toda]] | [[vdep]] — team defensive value from frequent proxies |
| 2022 | [[generalized-vdep-euro-location-analysis\|GVDEP]] | [[rikuhei-umemoto\|Umemoto]] | [[gvdep]] — score-scaled weights; partial-observation analysis |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]] — credit for [[space-creation\|space created]] for a teammate |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] | [[calvin-yeung\|Yeung]] | [[nmstpp]] and [[hpus]] |
| 2023 | [[team-defense-positioning-counterfactuals\|DRSO]] | Umemoto | [[drso]] — per-defender counterfactual positioning |
| **2023** | **[[action-valuation-multi-agent-reinforcement-learning\|Multi-agent deep RL valuation]]** | **[[hiroshi-nakahara\|Nakahara]]** | **[[multi-agent-reinforcement-learning\|Per-player RL agents]]; [[action-supervision]]** |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] | Yeung | [[game-theory\|Game-theoretic]] shot decisions; [[xsot\|xSOT]] |

> **Corrected 2026-08-07.** Nakahara et al. (2023) was previously listed under "cited, not held". It is now held, making seven.

## Two Signatures — and One Paper That Fits Neither

### Change the target, not the model

Visible in four of seven: **goals are too rare to model, so model something else on the causal path.**

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

### The outlier: Nakahara et al.

> **Added 2026-08-07.**

The RL paper belongs to neither signature, and its departures are pointed.

- It **declines the target substitution.** The reward is the goal itself, plus a conceding penalty and terminal [[expected-possession-value|EPV]]. In a group whose defining move is "goals are too rare, model something else", this paper models goals.
- It **is counterfactual without a reference.** No predicted trajectory, no optimal grid vertex — a learned $Q$ over the whole [[action-space-design|action space]] yields values for unchosen actions directly.

That makes it a **third route** to per-player off-ball value alongside C-OBSO and DRSO, and the only one not requiring a baseline to difference against. See [[off-ball-value]].

## The Group's Own Metrics Do Not Agree

> **Added 2026-08-07 — the most consequential finding of this ingest.**

Nakahara et al. correlate their Q-values against [[c-obso|C-OBSO]] on **the same club, season and data provider**: $\rho = 0.182$, no relationship. Against [[obso|OBSO]]: $-0.305$.

Two Fujii-group metrics, both presented as measures of off-ball contribution, computed on the same 14 Yokohama players, are statistically unrelated.

The paper's explanation is that C-OBSO favours forwards while Q-values favour midfielders and defenders. That is plausible and partly self-undermining: if each ranks a different position group, neither measures off-ball contribution *as such*. See [[construct-validity]].

This is the first head-to-head between two off-ball metrics anywhere in the vault, and it is a same-group comparison — which is exactly the caveat noted below.

## Iteration Within the Group

**Each paper fixes a limitation the previous one named.** VDEP states three; GVDEP addresses all three. NMSTPP's stated gaps — no defensive actions, no player-level attribution — are addressed by VDEP and C-OBSO. VDEP's stated inability to individuate is addressed by DRSO. C-OBSO's inability to value players who never receive the ball is addressed by Nakahara et al.

That is rare in this literature, where papers more often propose alternatives than repair predecessors. It also means the group's own work is the closest thing the vault has to [[action-valuation-frameworks-compared|cross-framework comparison]] — GVDEP against VDEP, and now Q-values against C-OBSO, on identical data — though same-group comparison is not an independent benchmark.

**And the iteration has a blind spot.** Nakahara et al.'s independent agents do not model each other; [[optimal-decisions-shot-taking-situations|Yeung & Fujii]] model a best-responding opponent explicitly. Two papers from one group in consecutive years, on strategic interaction in football, neither citing the other. See [[multi-agent-reinforcement-learning]].

## Methodological Range

| Approach | Work |
|---|---|
| Supervised classification on engineered state | [[vdep]], [[gvdep]] |
| [[transformer]] [[point-process\|point process]] | [[nmstpp]] |
| [[graph-neural-network\|Graph]] [[variational-autoencoder\|VAE]] trajectory prediction | [[c-obso]] |
| [[theory-based-modelling\|Theory-based]] geometry + MLP + [[game-theory\|game theory]] | [[xsot]] |
| **[[temporal-difference-learning\|TD]] [[reinforcement-learning\|RL]] with [[gated-recurrent-unit\|GRU]] and [[action-supervision]]** | **Nakahara et al.** |
| **Physical surface + counterfactual search, no ML** | **[[drso]]** |

No single architecture recurs. What recurs is **prediction or physics first, metric derived downstream** — and Nakahara et al. now break even that: the metric *is* the model output, learned directly against reward. Of the seven, it is the only one where value is not derived from something else.

DRSO is the outlier at the opposite end: **no machine learning at all**, deliberately, on the argument that interpretability is what makes advice actionable for coaches.

## A Caution

Two of the group's papers ([[c-obso]], [[drso]]) set PPCF parameters $\sigma = 0.45$, $\lambda = 4.3$ citing [[beyond-expected-goals|Spearman (2018)]], which fits $s = 0.54$, $\lambda = 3.99$. A citation error propagating through the line — see [[obso]].

## Cited, Not Held

- **Teranishi, Fujii & Takeda (2020)**, IEEE GCCE pp. 124–125 — earliest of the trajectory line.
- **Fujii (2021)**, *Data-driven analysis for understanding team sports behaviors*, arXiv:2102.07545 — the group's own survey.
- **Fujii, Tsutsui, Scott, Nakahara, Takeishi & Kawahara (2023)**, *Adaptive action supervision in RL from real-world multi-agent demonstrations*, arXiv:2305.13030 — the dedicated treatment of [[action-supervision]], and the highest-value acquisition target from this ingest.
- **Fujii, Takeuchi, Kuribayashi et al. (2022)**, *Estimating counterfactual treatment outcomes over time in complex multi-agent scenarios*, arXiv:2206.01900.
- **Scott, Fujii & Onishi (2022)**, *How does AI play football?* — RL against real-world strategies; bears on the [[google-research-football|simulator transfer]] question.

## See Also

- [[vdep]] · [[gvdep]] · [[drso]] · [[c-obso]] · [[nmstpp]] · [[xsot]] · [[hpus]] · [[obso]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[temporal-difference-learning]] · [[reinforcement-learning]]
- [[rare-event-proxy-targets]] · [[counterfactual-baseline]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[construct-validity]]
- [[game-theory]] · [[theory-based-modelling]] · [[interpretability]]
- [[calvin-yeung]] · [[kosuke-toda]] · [[rikuhei-umemoto]] · [[masakiyo-teranishi]] · [[hiroshi-nakahara]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-kushiro]] · [[tony-sit]]
- [[nagoya-university]] · [[kyoto-university]] · [[data-stadium]] · [[william-spearman]] · [[google-research-football]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[optimal-decisions-shot-taking-situations|SPC Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
