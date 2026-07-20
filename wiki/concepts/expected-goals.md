---
title: "Expected Goals (xG)"
type: concept
tags: [sports-analytics, statistics, machine-learning, evaluation]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 30%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Expected Goals (xG)

Expected Goals (xG) is a statistical metric that estimates the probability of a shot resulting in a goal, based on features of the shot opportunity (location, angle, body part, preceding action, defensive pressure, etc.).

## How It Works

For each goal attempt, an xG model predicts the probability $P(\text{goal} | \text{shot features})$ using a classifier trained on historical shot data. A shot from the penalty spot might have xG ≈ 0.76, while a long-range effort might have xG ≈ 0.03. Summing a player's xG over a season gives their expected goal tally, enabling comparison with their actual goals.

## Limitations

The [[evaluating-football-player-actions|VAEP paper (Decroos et al., 2019)]] identifies three limitations of xG-based player evaluation:

1. **Shot-centric:** Only values the final shot, ignoring all preceding actions (passes, dribbles, take-ons) that created the opportunity.
2. **Context-blind:** Assigns fixed values based on shot location without considering the full game state (preceding actions, defensive shape, speed of play).
3. **Immediate only:** Does not capture longer-term effects of actions several steps before a shot.

## Relation to VAEP

xG is a special case of [[vaep]]: computing the xG of a shot is equivalent to estimating $P_{scores}$ at the game state immediately before the shot attempt. VAEP generalises this to all 21 [[spadl]] action types — passes, dribbles, tackles, clearances — not just shots.

## Widespread Adoption

Despite its limitations, xG has become the most widely used advanced metric in soccer analytics, adopted by broadcasters, clubs, and media. It represents the first widely successful application of machine learning to soccer player evaluation.

## See Also

- [[vaep]]
- [[spadl]]
- [[evaluating-football-player-actions|Source Summary]]
- [[game-state-reconstruction]]
