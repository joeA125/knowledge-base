---
title: "Nagoya University"
type: entity
tags: [entity, organisation, university, ai-research, sports-analytics, off-ball, defensive-valuation, event-prediction, reinforcement-learning, counterfactual, domain-adaptation, animal-behaviour]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 41%
  generated: 5%
  imported: 0%
  ambiguous: 4%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Nagoya University

Japanese research university. Its Graduate School of Informatics is the base of [[keisuke-fujii]], and through him the institutional home of the vault's Japanese sports-analytics line — **the largest single-institution cluster here by a wide margin.**

> **Corrected 2026-08-07.** This page stated that two vault sources originate here. Accurate when written and badly drifted since. **Eight** now do.

## The Eight

| Source | Year | Approach | Target |
|---|---|---|---|
| [[football-defence-evaluation-vdep\|VDEP]] | 2022 | Event classification with off-ball state | **Defensive** value from frequent proxies |
| [[generalized-vdep-euro-location-analysis\|GVDEP]] | 2022 | Score-scaled reweighting; input sweep | Defensive value, partial observation |
| [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | 2022/23 | GVRNN [[trajectory-prediction]] + counterfactual | [[space-creation\|Space created]] for a teammate |
| [[transformer-point-process-football-event-modelling\|NMSTPP]] | 2023 | [[transformer]] [[point-process\|point process]] | Forecasting event time, zone, action |
| [[team-defense-positioning-counterfactuals\|DRSO]] | 2023 | Geometric counterfactual, **no ML** | Per-defender positioning |
| [[action-valuation-multi-agent-reinforcement-learning\|Multi-agent deep RL]] | 2023 | [[temporal-difference-learning\|TD]] RL, ten agents, **inverse** | On- and off-ball value at every timestep |
| [[adaptive-action-supervision-multi-agent-rl\|Adaptive action supervision]] | 2023 | [[deep-q-network\|DDQN]] + [[dynamic-time-warping\|DTW]], **forward** | A method, not a metric |
| [[optimal-decisions-shot-taking-situations\|SPC framework]] | 2024 | [[game-theory\|Game theory]] + [[theory-based-modelling\|theory-based]] geometry | Optimal shot-or-pass decision |

## Two Registers, One Institution

> **Added 2026-08-07.** The eighth source makes a division visible that the previous seven obscured.

Seven papers have Fujii as **senior** author with Nagoya students leading. One has him **first**, with co-authors from [[university-of-tokyo|Tokyo]], [[osaka-university|Osaka]] and RIKEN who appear on no applied football paper here.

| | Applied line | Methodological line |
|---|---|---|
| Papers | VDEP, GVDEP, C-OBSO, NMSTPP, DRSO, SPC, Nakahara et al. | Fujii et al. (2023) |
| Subject | Football | **Biological multi-agents generally** |
| Football is | The object of study | **A test case** |
| Deliverable | A metric | A method |

The methodological paper's two experiments are a predator-prey chase task and football, and it closes by proposing animal behaviour as the more valuable direction. **Its football results are demonstrations, not claims about football** — see [[yoshinobu-kawahara]] and [[kazushi-tsutsui]], whose collaborative-hunting line supplies the other experiment.

## The Signature, and Where It Breaks Twice

Six of the eight come at football through **prediction or physics rather than valuation**, deriving metrics downstream from a model built for something else. That distinguishes the group from the Leuven ([[vaep]]) and Barcelona ([[expected-value-possession-framework|EPV]]) lines.

**Nakahara et al. break it one way** — the value function is not derived from a predictive model, it *is* the model, trained against reward.

**Fujii et al. break it another** — there is no metric at all. It is the only held source here whose deliverable is a method, and the only one that goes **forward**, building an environment and generating behaviour rather than estimating from data. See [[multi-agent-reinforcement-learning]].

## The Cluster Can Check Itself, and the Check Is Unflattering

The concentration has an underrated consequence: this is the only institution in the vault with enough overlapping work on one dataset to **compare its own metrics**, and the comparison has now been run once.

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] correlate their Q-values against [[c-obso|C-OBSO]] on the same club, season and [[data-stadium|provider]]: $\rho = 0.182$. Two Nagoya metrics, both for off-ball contribution, unrelated. See [[construct-validity]] and [[off-ball-value]].

**A second internal inconsistency surfaced with the eighth source.** Nakahara et al. and Fujii et al. use different [[action-space-design|action spaces]] (14 against 12) on the same data with overlapping authors, and different regularisers ($L_1$ against $L_2$) with the same small-data justification. Neither cites the other's choice.

Concentration makes comparison *possible* and apparently does not make it *happen*. The group's papers repair each other's stated limitations reliably — see [[keisuke-fujii]] — but do not reconcile their unstated design choices.

## Funding Structure

Fujii holds joint appointments at Nagoya, the RIKEN Center for Advanced Intelligence Project, and JST PRESTO — typical of Japanese research funding, where a university post combines with national-institute affiliation and a fixed-term grant. The two 2023 RL papers acknowledge JSPS KAKENHI 20H04075, 21H04892, 21H05300, 23H03282 and JST PRESTO JPMJPR20CA.

## People

[[keisuke-fujii]] (author on all eight) · [[hiroshi-nakahara]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[atom-scott]]. [[kosuke-toda]] leads VDEP from [[kyoto-university]]; [[naoya-takeishi]] and [[yoshinobu-kawahara]] join from Tokyo and Osaka via RIKEN.

## See Also

- [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[calvin-yeung]] · [[hiroshi-nakahara]] · [[rikuhei-umemoto]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[atom-scott]] · [[naoya-takeishi]] · [[yoshinobu-kawahara]]
- [[vdep]] · [[gvdep]] · [[c-obso]] · [[drso]] · [[nmstpp]] · [[hpus]] · [[xsot]] · [[multi-agent-reinforcement-learning]] · [[domain-adaptation]]
- [[off-ball-value]] · [[construct-validity]] · [[action-space-design]] · [[action-valuation-frameworks-compared]]
- [[kyoto-university]] · [[university-of-tokyo]] · [[osaka-university]] · [[data-stadium]] · [[google-research-football]] · [[nfootball]]
- [[football-defence-evaluation-vdep|VDEP]] · [[transformer-point-process-football-event-modelling|NMSTPP]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
