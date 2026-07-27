---
title: "Elo Rating System"
type: concept
tags: [statistics, ranking-system, paired-comparison, gaming, sports-analytics]
sources: [raw/papers/bayesian-true-skill-rating.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.9
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-07-27
---

# Elo Rating System

The Elo rating system, developed by Arpad Elo in 1959 and adopted by the World Chess Federation (FIDE) in 1970, is a method for estimating relative skill in two-player games.

## Model

Each player $i$ has a skill rating $s_i$. In a game, each player exhibits performance $p_i \sim \mathcal{N}(s_i, \beta^2)$. The probability that player 1 wins is:

$$P(p_1 > p_2 | s_1, s_2) = \Phi\left(\frac{s_1 - s_2}{\sqrt{2}\beta}\right)$$

After each game, ratings are updated by a linearised rule controlled by a $K$-factor ($\alpha\beta\sqrt{\pi}$), which determines the weight of new evidence vs. old estimates.

## Variants

- **Gaussian variant:** Thurstone Case V model (as described above).
- **Logistic variant:** the [[bradley-terry-model]]; argued to be a better fit for chess data. Most modern Elo implementations use this variant.

Elo is best understood as an *online update rule* applied to a paired-comparison model, rather than as a model in its own right — the model is Thurstone or Bradley-Terry, and Elo is the incremental estimator.

## Limitations

- Point estimate only — no uncertainty tracking (ratings are "provisional" until enough games are played).
- No native support for teams or multi-player games (requires heuristics like "duelling").
- No explicit draw model.
- No representation of structural advantage attached to a *role* rather than a competitor — home advantage, playing white, defending an aerial duel.
- Slow convergence compared to Bayesian alternatives.

These limitations motivated the development of [[glicko-rating-system]] and [[trueskill]].

## Use in Football

Elo and its descendants appear in football analytics as **context features** rather than as ends in themselves. Dinsdale & Gallagher (2022) use Elo ratings of clubs and leagues to forecast a transferred player's early output, and the same role is played by Glicko-2 ratings in [[league-strength-rating]].

The purpose in both cases is to make performance comparable across competitions of unequal standard — a prerequisite for [[transfer-performance-prediction]], since raw per-90 metrics from different leagues are not on a common scale.

## See Also

- [[trueskill]] · [[glicko-rating-system]] · [[bradley-terry-model]]
- [[league-strength-rating]] · [[duel-skill-rating]]
- [[transfer-performance-prediction]]
