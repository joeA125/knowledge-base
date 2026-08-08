---
title: "Keisuke Fujii"
type: entity
tags: [person, researcher, ai-research, university, sports-analytics, defensive-valuation, off-ball, event-prediction, game-theory, proxy-target, counterfactual, reinforcement-learning, multi-agent, domain-adaptation, optical-tracking-data, animal-behaviour]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-08-07
---

# Keisuke Fujii

Researcher at the Graduate School of Informatics, [[nagoya-university]], with affiliations at the RIKEN Center for Advanced Intelligence Project and JST PRESTO. **Author on eight held sources** — more than twice anyone else in this vault.

## Eight Primary Sources

| Year | Work | Lead author | Contribution |
|---|---|---|---|
| 2022 | [[football-defence-evaluation-vdep\|VDEP]] | [[kosuke-toda\|Toda]] | [[vdep]] — team defensive value from frequent proxies |
| 2022 | [[generalized-vdep-euro-location-analysis\|GVDEP]] | [[rikuhei-umemoto\|Umemoto]] | [[gvdep]] — score-scaled weights; partial-observation analysis |
| 2022/23 | [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]] — credit for [[space-creation\|space created]] |
| 2023 | [[transformer-point-process-football-event-modelling\|NMSTPP]] | [[calvin-yeung\|Yeung]] | [[nmstpp]] and [[hpus]] |
| 2023 | [[team-defense-positioning-counterfactuals\|DRSO]] | Umemoto | [[drso]] — per-defender counterfactual positioning |
| 2023 | [[action-valuation-multi-agent-reinforcement-learning\|Multi-agent deep RL valuation]] | [[hiroshi-nakahara\|Nakahara]] | [[multi-agent-reinforcement-learning\|Per-player RL agents]]; [[action-supervision]] |
| **2023** | **[[adaptive-action-supervision-multi-agent-rl\|Adaptive action supervision]]** | **Fujii — *first author*** | **[[domain-adaptation\|Real-to-Sim]]; [[dynamic-time-warping\|DTW]]-adaptive supervision; [[nfootball]]** |
| 2024 | [[optimal-decisions-shot-taking-situations\|SPC framework]] | Yeung | [[game-theory\|Game-theoretic]] shot decisions; [[xsot\|xSOT]] |

> **Corrected 2026-08-07, twice.** Nakahara et al. moved from cited-not-held to held (six → seven). Later the same day arXiv:2305.13030 was acquired (seven → eight).

## The First-Authorship Is the Tell

Seven of the eight have Fujii as **senior** author. This one has him **first**, with a partly different collaborator set — [[atom-scott]], [[naoya-takeishi]] and [[yoshinobu-kawahara]], none of whom appear on any applied football-metric paper the vault holds.

That marks a genuine division in the programme, and it changes how the paper should be read.

| | The applied line | The methodological line |
|---|---|---|
| Papers | VDEP, GVDEP, C-OBSO, NMSTPP, DRSO, SPC, Nakahara et al. | **Fujii et al. (2023)** |
| Fujii's role | Senior author | **First author** |
| Co-authors | Nagoya students | Nagoya **+ [[university-of-tokyo\|Tokyo]] + [[osaka-university\|Osaka]] + RIKEN** |
| Subject | Football | **Biological multi-agents generally** |
| Football is | The object of study | **A test case** |

The 2023 paper's two experiments are a predator-prey chase task and football, and it closes by proposing **animal behaviour** as the more scientifically valuable direction. Its football results are demonstrations of a method, not claims about football — and should not be read the way [[c-obso|C-OBSO]] or [[vdep|VDEP]] can be. See [[yoshinobu-kawahara]].

## Two Signatures — and Two Papers That Fit Neither

### Change the target, not the model

Visible in four of eight: **goals are too rare to model, so model something else on the causal path.**

| Work | Rare target abandoned | Proxy adopted |
|---|---|---|
| [[vdep]], [[gvdep]] | Goals conceded | Ball recovery, effective attack |
| [[hpus]] | Goals | Possession dynamics — **no goal data at any stage** |
| [[xsot]] | Goal | Shot on target — "the minimum requirement" |

Empirically justified from inside the group's own work: VDEP measures [[vaep|VAEP's]] conceding classifier at F1 = 0.000, GVDEP at 0.08–0.15. See [[rare-event-proxy-targets]].

**The signature reappears in an unexpected place.** Fujii et al. add a **shot reward of +1** to their RL reward function explicitly "because the goal reward was sparse and limited" — the same move, executed inside a reward function rather than a classifier target. It is the most portable idea the group has, and they apply it without comment.

### Counterfactual on one named agent

| Work | Reference | Intervenes on |
|---|---|---|
| [[c-obso]] | Predicted "league average" trajectory | An attacker's movement |
| [[drso]] | Best reachable grid vertex | A defender's position |

Both build on [[obso|OBSO]] rather than event classification. See [[counterfactual-baseline]].

### The two outliers

**Nakahara et al.** declines the target substitution (the reward *is* the goal) and is counterfactual *without a reference* — a learned $Q$ over the whole [[action-space-design|action space]]. A third route to per-player off-ball value.

**Fujii et al. (2023)** is neither valuation nor counterfactual. It is the only held source in this group whose deliverable is **a method, not a metric**, and the only one that goes **forward** — building an environment and generating behaviour rather than estimating from data. See [[multi-agent-reinforcement-learning]].

## The Group's Own Metrics Do Not Agree

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] correlate their Q-values against [[c-obso|C-OBSO]] on **the same club, season and data provider**: $\rho = 0.182$, no relationship. Against [[obso|OBSO]]: $-0.305$.

Two Fujii-group metrics, both presented as measures of off-ball contribution, computed on the same 14 Yokohama players, statistically unrelated. The paper's explanation — C-OBSO favours forwards, Q-values midfielders and defenders — is plausible and partly self-undermining. See [[construct-validity]].

## Iteration Within the Group, and Where It Stops

**Each paper fixes a limitation the previous one named.** VDEP states three; GVDEP addresses all three. NMSTPP's gaps are addressed by VDEP and C-OBSO. VDEP's inability to individuate is addressed by DRSO. C-OBSO's inability to value players who never receive the ball is addressed by Nakahara et al.

That is rare in this literature, where papers more often propose alternatives than repair predecessors.

**But the iteration does not extend to parameters.** Fujii et al. propose [[action-supervision]] and **report no value for its weight $\lambda_1$**; Nakahara et al. borrow the loss and report $\lambda_1 = 0.01$ with a two-point ablation. The methodological paper is *less* specific about its own mechanism than the applied one that cites it. The vault predicted the reverse. See [[free-parameters-load-bearing]].

**And the interaction blind spot persists.** Nakahara et al.'s independent agents do not model each other; Fujii et al.'s decentralised agents do not either, and when they test a *centralised* alternative (CDS) it changes nothing. Meanwhile [[optimal-decisions-shot-taking-situations|Yeung & Fujii]] model a best-responding opponent explicitly. Three papers, one senior author, no cross-citation.

## Methodological Range

| Approach | Work |
|---|---|
| Supervised classification on engineered state | [[vdep]], [[gvdep]] |
| [[transformer]] [[point-process\|point process]] | [[nmstpp]] |
| [[graph-neural-network\|Graph]] [[variational-autoencoder\|VAE]] trajectory prediction | [[c-obso]] |
| [[theory-based-modelling\|Theory-based]] geometry + MLP + [[game-theory\|game theory]] | [[xsot]] |
| [[temporal-difference-learning\|TD]] RL with [[gated-recurrent-unit\|GRU]] and [[action-supervision]] | Nakahara et al. |
| **[[deep-q-network\|DDQN]] + [[dynamic-time-warping\|DTW]]-adaptive supervision in a custom simulator** | **Fujii et al. (2023)** |
| **Physical surface + counterfactual search, no ML** | **[[drso]]** |

No single architecture recurs. What recurs is **prediction or physics first, metric derived downstream** — broken by Nakahara et al. (value learned directly against reward) and by Fujii et al. (no metric at all).

## A Caution

Two of the group's papers ([[c-obso]], [[drso]]) set PPCF parameters $\sigma = 0.45$, $\lambda = 4.3$ citing [[beyond-expected-goals|Spearman (2018)]], which fits $s = 0.54$, $\lambda = 3.99$. A citation error propagating through the line — see [[obso]].

## Cited, Not Held

- **Scott, Fujii & Onishi (2022)**, *How does AI play football?* — RL against real-world football strategies. **Now the highest-value acquisition target** in this area; see [[atom-scott]].
- **Fujii (2021)**, *Data-driven analysis for understanding team sports behaviors*, arXiv:2102.07545 — the group's own survey.
- **Fujii, Takeishi, Kawahara & Takeda (2020)**, *Policy learning with partial observation and mechanical constraints*, arXiv:2007.03155.
- **Fujii, Takeishi, Tsutsui et al. (2021)**, *Learning interaction rules from multi-animal trajectories*, NeurIPS 34.
- **Fujii, Takeuchi, Kuribayashi et al. (2022)**, *Estimating counterfactual treatment outcomes over time in complex multi-agent scenarios*, arXiv:2206.01900.
- **Tsutsui, Takeda & Fujii (2023)**, *Synergizing deep RL and biological pursuit behavioral rule*, ICML workshop.
- **Ding, Takeda & Fujii (2022)**, *Deep RL in a racket sport for player evaluation* — the group's only non-football RL valuation work.
- **Teranishi, Fujii & Takeda (2020)**, IEEE GCCE pp. 124–125.

## See Also

- [[vdep]] · [[gvdep]] · [[drso]] · [[c-obso]] · [[nmstpp]] · [[xsot]] · [[hpus]] · [[obso]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[temporal-difference-learning]] · [[reinforcement-learning]] · [[deep-q-network]]
- [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-reward-tradeoff]] · [[nfootball]] · [[google-research-football]]
- [[rare-event-proxy-targets]] · [[counterfactual-baseline]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[construct-validity]]
- [[calvin-yeung]] · [[kosuke-toda]] · [[rikuhei-umemoto]] · [[masakiyo-teranishi]] · [[hiroshi-nakahara]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[atom-scott]] · [[naoya-takeishi]] · [[yoshinobu-kawahara]]
- [[nagoya-university]] · [[kyoto-university]] · [[university-of-tokyo]] · [[osaka-university]] · [[data-stadium]] · [[william-spearman]]
- [[football-defence-evaluation-vdep|VDEP]] · [[generalized-vdep-euro-location-analysis|GVDEP]] · [[team-defense-positioning-counterfactuals|DRSO]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] · [[transformer-point-process-football-event-modelling|NMSTPP]] · [[optimal-decisions-shot-taking-situations|SPC]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
