---
title: "Glicko Rating System"
type: concept
tags: [bayesian, statistics, ranking-system, paired-comparison, gaming, sports-analytics, duel-analysis, inference]
sources: [raw/papers/bayesian-true-skill-rating.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-05-08
updated: 2026-07-27
---

# Glicko Rating System

Glicko is a Bayesian rating system developed by [[mark-glickman]] (1999) that extends the [[elo-rating-system]] by modelling belief about a player's skill as a Gaussian distribution — a mean $\mu$ and a **rating deviation** $\phi$ — rather than a single point estimate.

## Key Advance over Elo

Tracking uncertainty makes two things possible that Elo handles only with heuristics.

**Adaptive update size.** A result carries more information when either party's rating is uncertain, so the update is scaled accordingly. Elo's fixed $K$-factor cannot express this and requires ad-hoc "provisional rating" rules instead.

**Uncertainty growth during inactivity.** $\phi$ widens over time without play and narrows with games played, so a returning competitor's rating moves quickly rather than being anchored to stale evidence.

## Glicko-2

Glicko-2 adds a third parameter, **volatility** $\sigma$, capturing how erratic a competitor's results are. Two players with the same mean and the same uncertainty may differ in consistency, and volatility lets the system distinguish a stable performer from a streaky one — updating the latter more readily.

The rating update takes the form

$$\mu' = \mu + \phi'^2\, g(\phi_j)\left(s_j - E(\mu, \mu_j, \phi_j)\right)$$

where $s_j$ is the observed score, $E(\cdot)$ the expected score, and $g(\phi_j)$ a weight that shrinks the update when the *opponent's* rating is uncertain.

## Modifications for Asymmetric Contests

Glicko inherits [[bradley-terry-model|Bradley-Terry's]] **symmetry assumption**: the two sides differ only in latent strength. Real contests often have a structural advantage attached to a role rather than a competitor — home advantage, playing white, or facing your own goal in an aerial duel.

[[andrei-shelopugin|Shelopugin]] and [[alexander-sirotkin|Sirotkin]] address this by adding an advantage term inside the expectation:

$$\mu' = \mu + \phi'^2 g(\phi_j)\left(s_j - E(\mu + a, \mu_j, \phi_j)\right)$$

The advantaged party is *expected* to win more often, so gains less from winning and loses more from losing. The generalisation that matters is that $a$ is **learned from context** by a separate model rather than fitted as one global constant — it varies with pitch location, set-piece type, and the number of players nearby. See [[duel-skill-rating]].

## Applications Beyond Games

Glicko-2 was designed for chess and online play, but its structure suits any repeated pairwise contest. Two football applications appear in this vault, at different scales:

| Application | Competitors | Contest | Modification |
|---|---|---|---|
| [[duel-skill-rating]] | Individual players | An aerial or ground duel | Context-learned advantage term |
| [[league-strength-rating]] | Clubs | A match | League strength as mean of top-$n$ clubs |

The second is the more consequential downstream: club ratings are what make player metrics comparable across competitions, and hence what makes [[transfer-performance-prediction]] tractable.

Both applications discard the uncertainty they compute — downstream models consume $\mu$ as a point estimate and ignore $\phi$. That is a missed opportunity, since knowing *which* ratings are reliable is exactly the kind of information a recruitment model should propagate.

## Relation to TrueSkill

[[trueskill]] builds on Glicko's Gaussian skill beliefs and extends them to teams and multi-player competitions via [[factor-graph]] inference, using [[expectation-propagation|approximate message passing]] rather than Glicko's closed-form update. Glicko remains simpler and adequate wherever contests are genuinely pairwise.

## See Also

- [[elo-rating-system]] · [[trueskill]] · [[bradley-terry-model]]
- [[duel-skill-rating]] · [[league-strength-rating]]
- [[mark-glickman]] · [[andrei-shelopugin]] · [[alexander-sirotkin]]
- [[bayesian-inference]] · [[gaussian-density-filtering]]
- [[epv-control-duel-skills-football|Source Summary]]
