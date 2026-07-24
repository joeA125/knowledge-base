---
title: "On-Ball Value (OBV)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, event-stream-data, evaluation]
sources: [raw/papers/eventgpt-player-impact-from-team-action-sequences.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# On-Ball Value (OBV)

On-Ball Value (StatsBomb, 2021) assigns a value to each on-ball action according to how it changes the expected outcome of the **current possession sequence**. It is StatsBomb's commercial entry in the [[action-valuation]] family, alongside [[vaep]], [[expected-threat|xT]] and [[expected-goals|xG]].

Being a vendor metric, its exact formulation is not fully public — this page reflects how it is described and used in [[eventgpt-player-impact-team-action-sequences|Lee, Hong et al. (2025)]] rather than a primary specification.

## Position Among Valuation Frameworks

| | Horizon | Models risk? |
|---|---|---|
| [[expected-goals\|xG]] | The shot itself | No |
| [[expected-threat\|xT]] | Current possession | No |
| **OBV** | Current possession | Implicitly (possession outcome) |
| [[vaep]] | Next $k=10$ actions, across turnovers | Yes |
| [[martingale-epv]] | End of possession | Implicitly |

OBV sits close to xT in horizon — bounded by the possession — but values a broader action set, and being commercial it is widely available in StatsBomb-supplied data, which partly explains its adoption in applied work over the more academically visible xT and VAEP.

## Residual OBV: Making Value Forward-Looking

The substantive contribution of [[eventgpt]] is not OBV itself but a reformulation of it.

Every metric in the table above is **retrospective**: it scores an action after observing it. The authors' objection is that value is then "applied as a post-hoc layer on completed event sequences… rather than co-learned with the sequential process that generates actions."

**Residual On-Ball Value** instead accumulates forward:

$$rOBV_t = \mathbb{E}\left[\sum_{\tau=t}^{T_{\text{episode}}} OBV_\tau \;\Big|\; \text{state at } t, \text{player } p_t\right]$$

Two consequences follow:

1. **It becomes a prediction target**, so a generative model can be trained to estimate it *while* learning the sequence — value and dynamics co-learned rather than layered.
2. **It captures downstream influence.** Because the sum runs to the end of the episode, rOBV reflects how a player's presence shapes what happens *next*, not merely the action they are about to take. That is the quantity a transfer question actually concerns.

The analogy to reinforcement learning is close: rOBV is essentially a **state-value function** $V(s)$ conditioned on player identity — expected cumulative future reward — where per-action OBV plays the role of immediate reward. The vault's other explicitly forward-looking construction, [[martingale-epv]], is the same idea with a [[martingale]] guarantee attached.

## Aggregating a Skewed Distribution

A practical wrinkle worth recording. Attackers have high-variance OBV distributions — many low-value actions, rare decisive ones — so averaging Monte Carlo samples with an arithmetic mean is dominated by the frequent low-value outcomes and understates their impact.

[[eventgpt]] therefore uses a **truncated mean over the top quartile** for attackers and an arithmetic mean for everyone else. This is a reasonable response to skew, but it is hand-chosen and position-dependent, which means values are **not directly comparable across positions**. Any metric aggregating heavy-tailed per-action values faces the same choice.

## See Also

- [[eventgpt]]
- [[action-valuation]]
- [[vaep]]
- [[expected-threat]]
- [[martingale-epv]]
- [[eventgpt-player-impact-team-action-sequences|Source Summary]]
