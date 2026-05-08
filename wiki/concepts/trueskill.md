---
title: "TrueSkill"
type: concept
tags: [bayesian, ranking-system, matchmaking, gaming, statistics]
sources: [raw/papers/bayesian-true-skill-rating.md]
confidence: 0.95
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-05-08
---

# TrueSkill

TrueSkill is a Bayesian skill rating system developed by [[microsoft-research]] (Herbrich, Minka & Graepel, 2006) that generalises the [[elo-rating-system]]. It is used for player matchmaking in online gaming, most notably on Xbox Live.

## Key Properties

- **Uncertainty tracking:** Each player's skill is a Gaussian belief $\mathcal{N}(\mu, \sigma^2)$, not a point estimate.
- **Draw modelling:** Explicit draw margin $\epsilon$ models the probability of tied outcomes.
- **Team support:** Individual skills are inferred from team results by modelling team performance as the sum of individual performances.
- **Multi-player support:** Handles arbitrary numbers of competing players/teams via pairwise team performance difference comparisons.
- **Fast convergence:** Approaches target skill in ~10 games for 8-player matches, near the information-theoretic limit.

## How It Works

1. Model the game as a [[factor-graph]] with variables for skills, performances, team performances, and performance differences.
2. Run [[approximate-message-passing]] (based on [[expectation-propagation]]) to compute posterior skill distributions.
3. Use [[gaussian-density-filtering]]: the posterior after each game becomes the prior for the next.

## Skill Display

The displayed TrueSkill rating is a conservative estimate: $\mu - 3\sigma$. This ensures leaderboard tops are populated only by players who are both highly skilled and well-measured.

## Matchmaking

Match quality is derived from draw probability relative to the maximum possible draw probability, aligning fair matchmaking with informative experimental design.

## See Also

- [[elo-rating-system]]
- [[glicko-rating-system]]
- [[factor-graph]]
- [[approximate-message-passing]]
- [[bayesian-inference]]
- [[bayesian-true-skill-rating|Source Summary]]
