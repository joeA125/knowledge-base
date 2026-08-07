---
title: "Hiroshi Nakahara"
type: entity
tags: [person, researcher, university, ai-research, sports-analytics, reinforcement-learning, multi-agent, off-ball, action-valuation, optical-tracking-data, single-source]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.7
provenance:
  extracted: 70%
  inferred: 25%
  generated: 4%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Hiroshi Nakahara

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Lead author of the vault's only genuine [[reinforcement-learning|reinforcement-learning]] framework for football valuation.

## Held Work

| Year | Work | Contribution |
|---|---|---|
| 2023 | [[action-valuation-multi-agent-reinforcement-learning\|Action Valuation of On- and Off-Ball Soccer Players]] | [[multi-agent-reinforcement-learning\|Per-player RL agents]]; [[action-supervision]]; [[off-ball-value\|off-ball]] valuation at every timestep |

Co-authors [[kazushi-tsutsui]], [[kazuya-takeda]] and [[keisuke-fujii]] — the standard [[nagoya-university|Nagoya]] configuration, with Fujii as senior author.

## Position in the Fujii Group

The group's work divides into two established halves: **change the target** ([[vdep]], [[gvdep]], [[hpus]], [[xsot]]) and **counterfactual on one named agent** ([[c-obso]], [[drso]]). See [[keisuke-fujii]].

Nakahara's paper belongs to neither cleanly, and that is what makes it notable.

- It does **not** change the target — goals are the reward, supplemented by [[expected-possession-value|EPV]] and a conceding penalty. The group's signature move, adopted in four of its other papers because goals are too rare to model, is declined here.
- It **is** counterfactual, but not by intervening on one agent against a reference. The counterfactuals come free, because a learned $Q$ is defined over the entire [[action-space-design|action space]].

So it is the group's only framework that gets counterfactual values **without a comparison baseline** — no predicted trajectory as in [[c-obso|C-OBSO]], no optimal grid vertex as in [[drso|DRSO]]. That is a genuinely third route, and it is set out on [[off-ball-value]].

## Also Credited

Nakahara appears as a co-author on **Fujii, Tsutsui, Scott, Nakahara, Takeishi & Kawahara (2023)**, *Adaptive action supervision in reinforcement learning from real-world multi-agent demonstrations*, arXiv:2305.13030 — cited by the held paper as the basis for its supervision loss, and **not held**. That paper would presumably answer the open $\lambda_1$ question raised on [[action-supervision]], and is the highest-value acquisition target arising from this ingest.

## A Note on Confidence

This page rests on a single source and states little beyond it. Nakahara's independent research profile, subsequent work, and whether the RL line continued are all unknown here. Marked `single-source` accordingly.

## See Also

- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[action-space-design]] · [[reinforcement-learning]]
- [[off-ball-value]] · [[action-valuation]] · [[c-obso]] · [[construct-validity]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[calvin-yeung]]
- [[nagoya-university]] · [[data-stadium]] · [[google-research-football]]
- [[action-valuation-multi-agent-reinforcement-learning|Source Summary]]
