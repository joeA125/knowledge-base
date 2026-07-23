---
title: "Expected Goals (xG)"
type: concept
tags: [sports-analytics, statistics, machine-learning, evaluation, action-valuation, player-evaluation]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.9
provenance:
  extracted: 65%
  inferred: 28%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-23
---

# Expected Goals (xG)

Expected Goals (xG) is a statistical metric that estimates the probability of a shot resulting in a goal, based on features of the shot opportunity (location, angle, body part, preceding action, defensive pressure, etc.).

## How It Works

For each goal attempt, an xG model predicts the probability $P(\text{goal} \mid \text{shot features})$ using a classifier trained on historical shot data. A shot from the penalty spot might have xG ≈ 0.76, while a long-range effort might have xG ≈ 0.03. Summing a player's xG over a season gives their expected goal tally, enabling comparison with their actual goals.

## Limitations

The [[evaluating-football-player-actions|VAEP paper (Decroos et al., 2019)]] identifies three limitations of xG-based player evaluation:

1. **Shot-centric:** Only values the final shot, ignoring all preceding actions (passes, dribbles, take-ons) that created the opportunity.
2. **Context-blind:** Assigns fixed values based on shot location without considering the full game state (preceding actions, defensive shape, speed of play).
3. **Immediate only:** Does not capture longer-term effects of actions several steps before a shot.

Shots and assists together constitute **less than 1% of all on-the-ball actions** — the observation that motivates the whole [[action-valuation]] literature.

## A Building Block, Not Just a Competitor

xG is not merely superseded by broader frameworks; it is a *component* of them.

- In [[vaep]], xG is a special case: computing the xG of a shot is equivalent to estimating $P_{scores}$ at the game state immediately before the shot. VAEP generalises this to all 21 [[spadl]] action types.
- In [[expected-threat|xT]], xG appears explicitly inside the value recursion: $xT(z) = s_z \cdot xG(z) + m_z \sum_{z'} T_{z \to z'} xT(z')$. The term $s_z \cdot xG(z)$ is the immediate reward for shooting from zone $z$, and the whole model is built on top of it.

So a zone is "threatening" in xT precisely because shots taken from it, or reachable from it, have high xG. Improving the underlying xG model improves both.

## Widespread Adoption

Despite its limitations, xG has become the most widely used advanced metric in soccer analytics, adopted by broadcasters, clubs, and media. It represents the first widely successful application of machine learning to soccer player evaluation.

## See Also

- [[action-valuation]]
- [[vaep]]
- [[expected-threat]]
- [[spadl]]
- [[action-valuation-frameworks-compared]]
- [[evaluating-football-player-actions|VAEP Source Summary]]
- [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Comparison Summary]]
