---
title: "Duel Skill Rating"
type: concept
tags: [sports-analytics, duel-analysis, ranking-system, paired-comparison, bayesian, player-evaluation, sample-weighting, single-source]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.75
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Duel Skill Rating

Rating an individual footballer's ability to win aerial and ground duels, treating each duel as a paired comparison and inferring latent strength from outcomes. Developed by [[andrei-shelopugin|Shelopugin]] and Sirotkin (2023) and refined in the [[epv-control-duel-skills-football|EPV paper]].

## Why Win Percentage Fails

The obvious metric — proportion of duels won — is broken by **matchmaking**, and the failure is structural rather than statistical.

Coaches assign players to opponents. Strong aerial targets are marked by strong aerial defenders, especially at set pieces. Weak players contest weak opponents. The result is that win rates across the population **compress toward 50%**, and the players at the extremes look most average of all. A high win rate can then mean either genuine dominance or a soft schedule, and the statistic cannot distinguish them.

This is the same objection [[elo-rating-system|Elo]] was invented to answer in chess, and the remedy is the same: model the opponent.

## From Bradley-Terry to Glicko-2

The preceding approach, from practitioner [[garry-gelade]], applies the [[bradley-terry-model]] to duel outcomes. The paper credits it as a genuine advance — it accounts for opponent strength and produces ratings transferable across leagues, allowing comparison of players who never meet.

Two objections are raised:

1. **Bradley-Terry assumes symmetry.** Both contestants are treated as having equal a-priori chances, differing only in latent strength. Real duels are not symmetric — a defender facing his own goal during a defensive phase has a positional advantage independent of skill.
2. **It is not state of the art.** It carries no uncertainty and updates less efficiently than modern Bayesian alternatives.

The answer is a modified [[glicko-rating-system|Glicko-2]].

## The Advantage Term

Standard Glicko-2 updates the rating mean as

$$\mu' = \mu + \phi'^2 g(\phi_j)\left(s_j - E(\mu, \mu_j, \phi_j)\right)$$

The modification inserts a context-dependent advantage $a$ into the *expectation* for the advantaged party:

$$\mu' = \mu + \phi'^2 g(\phi_j)\left(s_j - E(\mu + a, \mu_j, \phi_j)\right)$$

A defender expected to win by virtue of position gains less for winning and loses more for losing. This is the same idea as home-advantage adjustment in team ratings, generalised to arbitrary context.

$a$ is not hand-set. It comes from a **separate [[gradient-boosting|LightGBM]] classifier** predicting duel outcome from context alone: duel coordinates, coordinates and type of the pass leading in (corner, free kick, open play), and number of opponents nearby. Every skill-related feature is deliberately excluded, so the model describes *situation difficulty for an average player* and nothing else.

## Two Leakage Problems

The paper's most careful section, and the part most worth borrowing.

**Phase leakage.** Defending teams win aerial duels more often than attacking teams. Without correction this is absorbed into player ratings rather than into the context model.

**Position leakage.** Centre-backs are genuinely strong in the air *and* contest a disproportionate share of their duels in the defensive phase, where the advantage term would otherwise credit the situation rather than them. Left uncorrected, the best aerial players in football would be systematically underrated.

The fix: partition players into **six position categories** — centre-backs, full-backs, midfielders, centre-forwards, wingers, goalkeepers — and train the context model only on duels where both contestants share a category. Same-category matchups isolate contextual difficulty from positional composition.

The instance loss is also divided by the player's frequency in the training set (see [[sample-weighting]]), so that heavily-featured players do not dominate the fit.

## Defining a Winner

Duel outcomes are not directly annotated, so the paper fixes a rule, applied to both duel types:

1. A player who is fouled is the winner.
2. Otherwise, whoever touches the ball first.
3. If neither touches it, whichever team gains possession.

Serviceable, but it makes ratings partly a function of the data provider's annotation conventions — a caveat that applies to any event-data rating system and is rarely stated.

## What the Ratings Look Like

Aerial ratings top out around 1762 (van Dijk) against a 1500 average; ground ratings compress into a narrower band, topping out near 1695. Two observations from the published tables:

- **Ground-duel ratings are almost entirely centre-backs**, which suggests the ground-duel definition captures defensive dispossession rather than the attacking take-on that [[vaep]] and [[expected-threat|xT]] would treat as the interesting one-versus-one.
- **Rating and raw win percentage diverge substantially.** Budkivskyi rates 8th aerially on a 56.2% win rate, while players with 77% win rates rate lower. That divergence is the whole point of the exercise, and is the clearest evidence the matchmaking correction is doing real work.

## See Also

- [[symmetrical-duel-valuation]] · [[bradley-terry-model]]
- [[glicko-rating-system]] · [[elo-rating-system]] · [[trueskill]]
- [[league-strength-rating]] · [[sample-weighting]]
- [[andrei-shelopugin]] · [[alexander-sirotkin]] · [[garry-gelade]]
- [[epv-control-duel-skills-football|Source Summary]]
