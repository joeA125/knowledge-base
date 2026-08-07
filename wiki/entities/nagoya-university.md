---
title: "Nagoya University"
type: entity
tags: [entity, organisation, university, ai-research, sports-analytics, off-ball, defensive-valuation, event-prediction, reinforcement-learning, counterfactual]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 48%
  inferred: 43%
  generated: 5%
  imported: 0%
  ambiguous: 4%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Nagoya University

Japanese research university. Its Graduate School of Informatics is the base of [[keisuke-fujii]], and through him the institutional home of the vault's Japanese sports-analytics line — **the largest single-institution cluster in this vault.**

> **Corrected 2026-08-07.** This page previously stated that two vault sources originate here. That was accurate when written and had drifted badly since. **Seven** now do.

## The Seven

| Source | Year | Approach | Target |
|---|---|---|---|
| [[football-defence-evaluation-vdep\|VDEP]] | 2022 | Event classification with off-ball state | **Defensive** value from frequent proxies |
| [[generalized-vdep-euro-location-analysis\|GVDEP]] | 2022 | Score-scaled reweighting; input sweep | Defensive value, partial observation |
| [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | 2022/23 | GVRNN [[trajectory-prediction]] + counterfactual | [[space-creation\|Space created]] for a teammate |
| [[transformer-point-process-football-event-modelling\|NMSTPP]] | 2023 | [[transformer]] [[point-process\|point process]] | Forecasting event time, zone, action |
| [[team-defense-positioning-counterfactuals\|DRSO]] | 2023 | Geometric counterfactual, **no ML** | Per-defender positioning |
| [[action-valuation-multi-agent-reinforcement-learning\|Multi-agent deep RL]] | 2023 | [[temporal-difference-learning\|TD]] [[reinforcement-learning\|RL]], ten agents | On- and off-ball value at every timestep |
| [[optimal-decisions-shot-taking-situations\|SPC framework]] | 2024 | [[game-theory\|Game theory]] + [[theory-based-modelling\|theory-based]] geometry | Optimal shot-or-pass decision |

## The Signature, and Where It Breaks

Six of the seven come at football through **prediction or physics rather than valuation**, deriving metrics downstream from a model built for something else. VDEP predicts recovery and penetration and combines them; NMSTPP predicts the next event and yields [[hpus]] from the forecasts; C-OBSO differences a trajectory prediction. That is a recognisable methodological signature, and it distinguishes the group from the Leuven ([[vaep]]) and Barcelona ([[expected-value-possession-framework|EPV framework]]) lines.

**The RL paper breaks it.** Its value function is not derived from a predictive model — it *is* the model, trained against reward. Of the seven, it is the only one where value is the direct training target rather than a downstream construction. See [[keisuke-fujii]].

## The Cluster's Own Metrics Disagree

The concentration has an underrated consequence: this is the only institution in the vault with enough overlapping work on one dataset to **check itself**, and the check has now been run once.

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] correlate their Q-values against [[c-obso|C-OBSO]] on the same club, season and [[data-stadium|provider]]: $\rho = 0.182$. Two Nagoya metrics, both for off-ball contribution, unrelated. See [[construct-validity]] and [[off-ball-value]].

That is the vault's only off-ball metric head-to-head, and it exists **because** the work is concentrated here — a benefit of the cluster and, since it is same-group, a limit on what the comparison establishes.

## Funding Structure

Fujii holds joint appointments at Nagoya, the RIKEN Center for Advanced Intelligence Project, and JST PRESTO — a structure typical of Japanese research funding, where a university post is combined with national-institute affiliation and a fixed-term project grant. The RL paper acknowledges JSPS KAKENHI grants 20H04075 and 21H05300 and JST PRESTO grant JPMJPR20CA.

## People

[[keisuke-fujii]] (senior author on all seven) · [[hiroshi-nakahara]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[kazushi-tsutsui]] · [[kazuya-takeda]]. [[kosuke-toda]] leads VDEP from [[kyoto-university]].

## See Also

- [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[calvin-yeung]] · [[hiroshi-nakahara]] · [[rikuhei-umemoto]] · [[kazushi-tsutsui]] · [[kazuya-takeda]]
- [[vdep]] · [[gvdep]] · [[c-obso]] · [[drso]] · [[nmstpp]] · [[hpus]] · [[xsot]] · [[multi-agent-reinforcement-learning]]
- [[off-ball-value]] · [[construct-validity]] · [[action-valuation-frameworks-compared]]
- [[kyoto-university]] · [[data-stadium]] · [[google-research-football]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
