---
title: "Elo Rating System"
type: concept
tags: [statistics, ranking-system, gaming]
sources: [raw/papers/bayesian-true-skill-rating.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-05-08
---

# Elo Rating System

The Elo rating system, developed by Arpad Elo in 1959 and adopted by the World Chess Federation (FIDE) in 1970, is a method for estimating relative skill in two-player games.

## Model

Each player $i$ has a skill rating $s_i$. In a game, each player exhibits performance $p_i \sim \mathcal{N}(s_i, \beta^2)$. The probability that player 1 wins is:

$$P(p_1 > p_2 | s_1, s_2) = \Phi\left(\frac{s_1 - s_2}{\sqrt{2}\beta}\right)$$

After each game, ratings are updated by a linearised rule controlled by a $K$-factor ($\alpha\beta\sqrt{\pi}$), which determines the weight of new evidence vs. old estimates.

## Variants

- **Gaussian variant:** Thurstone Case V model (as described above).
- **Logistic variant:** Bradley-Terry model; argued to be a better fit for chess data. Most modern Elo implementations use this variant.

## Limitations

- Point estimate only — no uncertainty tracking (ratings are "provisional" until enough games are played).
- No native support for teams or multi-player games (requires heuristics like "duelling").
- No explicit draw model.
- Slow convergence compared to Bayesian alternatives.

These limitations motivated the development of [[glicko-rating-system]] and [[trueskill]].

## See Also

- [[trueskill]]
- [[glicko-rating-system]]
