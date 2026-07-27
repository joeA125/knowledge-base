---
title: "Effective Playing Time"
type: concept
tags: [sports-analytics, evaluation, player-evaluation, event-stream-data, action-valuation, statistics, single-source]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Effective Playing Time

The total time the ball is actually in play, excluding stoppages — VAR reviews, substitutions, injury treatment, and set-piece dead time. Contrasted with **dirty playing time**, the clock duration of a match, which is what nearly every football metric is normalised on.

A nominal 90-minute match typically contains somewhere between 50 and 60 minutes of effective time, and the gap varies substantially by league, by team, and by scoreline.

## Why It Is Not a Detail

Normalising on clock time embeds two distortions that are easy to overlook because they affect all players in a match equally — but *not* all players across a season.

**Cross-team distortion.** Teams that lead often waste time; teams chasing games do not. A player at a defensive, lead-protecting side accumulates the same clock minutes as one at a high-tempo side while having materially fewer opportunities to act. Per-90 metrics silently penalise the former.

**Cross-league distortion.** Effective playing time differs systematically between competitions. Any metric compared across leagues on a per-90 basis is comparing rates measured against different denominators — which matters directly for the [[transfer-performance-prediction|cross-league forecasting]] this concept was introduced to support.

[[pass-carry-reward|PCR]] is normalised per 60 minutes of effective time for exactly this reason.

## The Subtler Role: Making Decay Behave

Effective time also fixes the denominator of [[temporal-discounting|the discount factor]], and the paper argues this is where it earns its place rather than in the normalisation.

The worked case is a won penalty. On the clock, a penalty may be taken two or three minutes after the foul — VAR check, protests, players clearing the box. Discounting at $\gamma = 0.95$ per second over wall-clock time would reduce the credit for winning it to essentially nothing, which is obviously wrong: winning a penalty is among the most valuable things a player can do.

In effective time the foul and the kick are adjacent, so the reward survives intact. Discounting over the wrong clock would have produced a model that systematically undervalues drawing fouls in dangerous areas.

The general principle: **when time enters a model as a measure of causal proximity, dead time must be removed**, because nothing causally relevant happens during it.

## Cost

The obvious one — it requires knowing when the ball is in play. [[event-stream-data|Event data]] gives this only approximately, since restarts are timestamped but the precise moment play stopped often is not. Any implementation involves reconstruction rules that are rarely documented, and the paper does not specify its own.

This makes effective-time figures less portable between data providers than clock minutes, which is presumably a large part of why the convention has not spread despite the argument for it being straightforward.

## See Also

- [[temporal-discounting]] · [[pass-carry-reward]]
- [[expected-possession-value]] · [[action-valuation]]
- [[event-stream-data]] · [[transfer-performance-prediction]]
- [[epv-control-duel-skills-football|Source Summary]]
