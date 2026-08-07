---
title: "Kazuya Takeda"
type: entity
tags: [person, researcher, university, sports-analytics, trajectory-prediction, imitation-learning, reinforcement-learning, multi-agent]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.7
provenance:
  extracted: 48%
  inferred: 40%
  generated: 5%
  imported: 0%
  ambiguous: 7%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Kazuya Takeda

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Co-author of two held sources.

| Work | Lead author | Contribution |
|---|---|---|
| [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | [[masakiyo-teranishi\|Teranishi]] | [[c-obso]]; GVRNN [[trajectory-prediction]] |
| [[action-valuation-multi-agent-reinforcement-learning\|Multi-agent deep RL valuation]] | [[hiroshi-nakahara\|Nakahara]] | [[multi-agent-reinforcement-learning\|Per-player agents]]; [[temporal-difference-learning\|TD]] value learning |

Also co-author of the earlier Teranishi, Fujii & Takeda (2020) trajectory-prediction work at IEEE GCCE — so he appears on both ends of the group's trajectory line — and, with [[keisuke-fujii]], on the cited-but-not-held *Policy learning with partial observation and mechanical constraints for multi-person modeling* (arXiv:2007.03155).

## The Movement-Modelling Thread

The recurrence across the group's **movement** papers, and absence from its event-classification ones ([[vdep]], [[nmstpp]]), suggests the movement side is his contribution — inference from co-authorship patterns rather than anything stated.

> **Strengthened 2026-08-07.** The RL paper fits the pattern and sharpens it. Its state is positions and velocities of all 23 entities; its action space is eight movement directions plus sprint labels gated on velocity thresholds; and its preprocessing follows the 10 Hz downsampling used in the 2020 arXiv:2007.03155 work, which the paper cites for exactly that. **Three held or cited works, one continuous thread: model where players move, then derive value from it.** See [[action-space-design]].

The thread also explains a design choice the RL paper does not justify. Movement is treated as the primary action vocabulary while on-ball behaviour is thinned to pass and shot — dribbling and trapping labels present in the data are discarded. That is the ordering a movement-modelling line would choose, and the reverse of what an event-stream line ([[vaep]], [[spadl]]) would.

**Note:** vault knowledge of this person comes from two primary sources and citations within them.

## See Also

- [[trajectory-prediction]] · [[c-obso]] · [[imitation-learning]] · [[action-space-design]] · [[optical-tracking-data]]
- [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[action-supervision]] · [[reinforcement-learning]]
- [[masakiyo-teranishi]] · [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[hiroshi-nakahara]]
- [[nagoya-university]] · [[data-stadium]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
