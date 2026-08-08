---
title: "Hiroshi Nakahara"
type: entity
tags: [person, researcher, university, ai-research, sports-analytics, reinforcement-learning, multi-agent, off-ball, action-valuation, optical-tracking-data, domain-adaptation]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.75
provenance:
  extracted: 68%
  inferred: 26%
  generated: 5%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Hiroshi Nakahara

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Lead author of the vault's first genuine [[reinforcement-learning|RL]] framework for football valuation, and co-author of the paper that supplies its method.

## Held Work

| Year | Work | Role | Contribution |
|---|---|---|---|
| 2023 | [[action-valuation-multi-agent-reinforcement-learning\|Action Valuation of On- and Off-Ball Soccer Players]] | **Lead** | [[multi-agent-reinforcement-learning\|Per-player RL agents]]; [[action-supervision]]; [[off-ball-value\|off-ball]] valuation at every timestep |
| 2023 | [[adaptive-action-supervision-multi-agent-rl\|Adaptive Action Supervision in RL]] | Co-author | [[domain-adaptation\|Real-to-Sim]] adaptation; [[dynamic-time-warping\|DTW]]-adaptive supervision |

> **Updated 2026-08-07.** The second paper was previously recorded on this page as the vault's highest-value acquisition target, on the expectation that it would settle the open $\lambda_1$ question. It has been acquired and **does not** — it reports no value for $\lambda_1$ at all. See [[free-parameters-load-bearing]].

The two were posted to arXiv within two weeks of each other (2305.17886 and 2305.13030), share a dataset, and face **opposite directions**.

## The Inverse Half of a Pair

| | Nakahara et al. (lead) | Fujii et al. (co-author) |
|---|---|---|
| Direction | **Inverse** — estimate from data | **Forward** — generate in a simulator |
| Environment | None; never acts | [[nfootball\|NFootball]] |
| Algorithm | SARSA, on-policy, no stabilisers | DDQN with [[deep-q-network\|full stabiliser stack]] |
| Supervision | Contemporaneous | **[[dynamic-time-warping\|DTW]]-adaptive** |
| Deliverable | A player-valuation metric | A method |
| Outcome | A metric that disagrees with [[c-obso\|C-OBSO]] | **Failed to reproduce demonstrated football** |

Reading them together resolves something neither states. **Nakahara et al. need no DTW alignment because they never roll out** — working inverse, agent and demonstration share timesteps by construction. Alignment is the price of going forward, and his own paper avoids paying it by declining the forward approach. See [[action-supervision]].

**The forward paper is the one that fails.** That is the strongest available justification for the design choice his own framework makes.

## Position in the Fujii Group

The group's applied work divides into **change the target** ([[vdep]], [[gvdep]], [[hpus]], [[xsot]]) and **counterfactual on one named agent** ([[c-obso]], [[drso]]). See [[keisuke-fujii]].

Nakahara's paper belongs to neither, and that is what makes it notable.

- It does **not** change the target — goals are the reward, supplemented by [[expected-possession-value|EPV]] and a conceding penalty. The group's signature move, adopted in four other papers because goals are too rare to model, is declined.
- It **is** counterfactual, but not by intervening on one agent against a reference. The counterfactuals come free, because a learned $Q$ is defined over the entire [[action-space-design|action space]].

So it is the group's only framework that gets counterfactual values **without a comparison baseline** — a genuinely third route, set out on [[off-ball-value]].

**Worth noting the contrast with the paper he co-authors.** Fujii et al. *do* apply the target substitution, adding a shot reward of +1 "because the goal reward was sparse and limited". The same author is on a paper that declines the group's signature move and one that applies it, months apart, on the same data. Neither comments.

## Action-Space Divergence

The two papers use different action sets on the same [[data-stadium|Data Stadium]] J1 2019 data: **14 actions** (his, with three sprint-state actions and one pass) against **12** (Fujii et al., with two pass types and no sprint state). Neither mentions the other's choice.

Not a contradiction — a simulator must decide a ball's trajectory where an inverse model reads pass type from the event stream — but it means the two frameworks cannot pose the same counterfactual questions. See [[action-space-design]].

## See Also

- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[deep-q-network]] · [[action-space-design]] · [[reinforcement-learning]]
- [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-reward-tradeoff]] · [[nfootball]] · [[google-research-football]]
- [[off-ball-value]] · [[action-valuation]] · [[c-obso]] · [[construct-validity]] · [[free-parameters-load-bearing]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[atom-scott]] · [[masakiyo-teranishi]] · [[rikuhei-umemoto]] · [[calvin-yeung]]
- [[nagoya-university]] · [[data-stadium]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
