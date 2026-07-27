---
title: "League and Club Strength Rating"
type: concept
tags: [sports-analytics, ranking-system, paired-comparison, bayesian, transfer-prediction, recruitment, statistics, evaluation]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# League and Club Strength Rating

Assigning latent strength ratings to clubs from match results, and to leagues from the ratings of their clubs — so that a player's output can be interpreted relative to the difficulty of the competition it was produced in.

The vault's rating-system pages ([[elo-rating-system]], [[glicko-rating-system]], [[trueskill]]) treat rating as an end in itself. Here it is **infrastructure**: the ratings exist to make player metrics cross-league comparable.

## The Problem It Solves

Raw player metrics are not comparable across competitions. The [[pass-carry-reward|PCR]] leaderboard makes this vivid — players from the Azerbaijani, Norwegian, Maltese and Northern Irish leagues appear at values comparable to Messi and Mbappé. Nobody believes those are equivalent performances.

The naive fix is a categorical feature per league, or per league-pair for transfers. Both fail:

- **Curse of dimensionality.** There are too few observed transfers between any specific pair of leagues to learn a coefficient.
- **Leagues change.** A league's strength is not a fixed property. Any static encoding is stale.

The rating approach solves both by putting all clubs on **one continuous, time-varying scale**. Transfers between never-before-observed league pairs are then handled by interpolating on the rating difference rather than by looking up an unseen category.

## Construction

[[andrei-shelopugin|Shelopugin]] and Sirotkin (2023) rate clubs with a modified [[glicko-rating-system|Glicko-2]] over match outcomes. **League strength is then defined as the mean rating of its top $n$ clubs**, with $n$ adjustable per league.

The top-$n$ definition rather than a full average is a deliberate choice, and a sensible one. League sizes differ, and the depth of a league's tail says little about the level a signing would face — a competition with four elite clubs and sixteen weak ones is not equivalent to one that is uniformly moderate, but their means may coincide.

Continental competition is what actually connects the scales: without cross-league fixtures there is no path linking an Azerbaijani club to an English one, and the ratings would be sixty disconnected scales rather than one. This dependence on inter-league matches is the method's structural constraint, and it means leagues whose clubs rarely qualify for continental play are rated on thin evidence.

## Derived Features

From these ratings the transfer model constructs:

- Rating of the player's current club, and of the destination club
- Difference between old and new **league** ratings
- **Mean rating of opponents faced** — two players with identical metrics may have played very different schedules

The last is the same strength-of-schedule correction that [[duel-skill-rating]] applies at the individual level, lifted to the season.

## The Temporal Constraint

Only ratings **available at the start of the season** are used. This is stated as a natural constraint but is the kind of discipline that is easy to violate accidentally: using end-of-season club ratings to predict that season's player performance leaks the outcome, since a club's rating rises partly *because* of the player being evaluated.

Worth noting as a general rule for any model with entity-level covariates that co-evolve with the target.

## Limitations

- **Club rating conflates squad with institution.** A club's rating reflects the players it currently has. Predicting a signing's output at a club rated 1700 assumes the club stays at 1700 — but it may be rated 1700 *because* of the player it is about to sell.
- **Style is not strength.** A league can be weak and yet stylistically demanding in ways that suit or ruin a particular player. The paper handles this separately, with league-style features such as mean league PCR, which is a partial answer at best.
- **Thin evidence for isolated leagues**, as above.
- **No uncertainty propagated.** Glicko-2 tracks rating deviation, but the downstream model consumes point estimates, discarding the information about which ratings are reliable.

## See Also

- [[glicko-rating-system]] · [[elo-rating-system]] · [[bradley-terry-model]]
- [[transfer-performance-prediction]] · [[recruitment]] · [[pass-carry-reward]]
- [[duel-skill-rating]] · [[selection-bias]]
- [[andrei-shelopugin]] · [[alexander-sirotkin]]
- [[epv-control-duel-skills-football|Source Summary]]
