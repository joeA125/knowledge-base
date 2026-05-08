---
title: "Glicko Rating System"
type: concept
tags: [bayesian, statistics, ranking-system, gaming]
sources: [raw/papers/bayesian-true-skill-rating.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 50%
  ambiguous: 10%
lifecycle: draft
created: 2026-05-08
updated: 2026-05-08
---

# Glicko Rating System

Glicko is a Bayesian rating system developed by Mark Glickman (1999) that extends the [[elo-rating-system]] by modelling the belief about a player's skill as a Gaussian distribution characterised by a mean $\mu$ and variance $\sigma^2$, rather than a single point estimate.

## Key Advance over Elo

By tracking uncertainty, Glicko addresses the problem of provisional ratings in the [[elo-rating-system]]. The uncertainty widens when a player is inactive and narrows as more games are played.

## Relation to TrueSkill

[[trueskill]] builds on Glicko's idea of Gaussian skill beliefs but extends it to handle teams and multi-player competitions via [[factor-graph]] inference.

## See Also

- [[elo-rating-system]]
- [[trueskill]]
