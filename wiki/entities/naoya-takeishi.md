---
title: "Naoya Takeishi"
type: entity
tags: [person, researcher, university, ai-research, machine-learning, reinforcement-learning, multi-agent, counterfactual, animal-behaviour, single-source]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.6
provenance:
  extracted: 50%
  inferred: 38%
  generated: 10%
  imported: 2%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Naoya Takeishi

Researcher at the Graduate School of Engineering, [[university-of-tokyo]], with an affiliation at the RIKEN Center for Advanced Intelligence Project. Co-author of [[adaptive-action-supervision-multi-agent-rl|Fujii et al. (2023)]].

## The Methodological Collaboration

Takeishi appears alongside [[keisuke-fujii]] and [[yoshinobu-kawahara]] on a recurring cluster of works — all **cited, not held** — that sit outside the football-analytics line and are more methodological in character:

| Work | Domain |
|---|---|
| Fujii, Takeishi, Kawahara & Takeda (2020), *Policy learning with partial observation and mechanical constraints for multi-person modeling*, arXiv:2007.03155 | Multi-person [[imitation-learning\|imitation]] |
| Fujii, Takeishi, Tsutsui et al. (2021), *Learning interaction rules from multi-animal trajectories via augmented behavioral models*, NeurIPS 34 | **Animal collective behaviour** |
| Fujii, Takeuchi, Kuribayashi, Takeishi, Kawahara & Takeda (2022), *Estimating counterfactual treatment outcomes over time in complex multi-agent scenarios*, arXiv:2206.01900 | [[counterfactual-simulation\|Counterfactual]] inference |

The pattern is consistent: **Takeishi and Kawahara appear on the group's work that generalises past sport** — animal behaviour, counterfactual estimation, multi-agent modelling as a class — and are absent from the applied football-metric papers ([[vdep]], [[c-obso]], [[nmstpp]], [[xsot]]).

That is inference from co-authorship rather than anything stated, but it fits the paper it appears on. [[adaptive-action-supervision-multi-agent-rl|Fujii et al. (2023)]] is explicitly framed around **biological multi-agents** in general — its two experiments are a predator-prey chase task and football — and closes by proposing animal-behaviour applications as the more scientifically valuable direction.

**Football is a data source in that paper, not the subject.** It has abundant tracking data and unknown governing dynamics, which is exactly what a Real-to-Sim method needs to be tested against. See [[domain-adaptation]].

## Note

Vault knowledge of this person comes from one source in which he is a middle author, plus citations within it. Nothing about his independent work is established here.

## See Also

- [[domain-adaptation]] · [[multi-agent-reinforcement-learning]] · [[imitation-learning]] · [[counterfactual-simulation]] · [[trajectory-prediction]]
- [[keisuke-fujii]] · [[yoshinobu-kawahara]] · [[atom-scott]] · [[kazushi-tsutsui]] · [[kazuya-takeda]]
- [[university-of-tokyo]] · [[nagoya-university]] · [[osaka-university]]
- [[adaptive-action-supervision-multi-agent-rl|Source Summary]]
