---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.75
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Control

A surface giving, for each location on the field, the probability that a given team would control the ball there. Built from tracking data, it turns 22 point positions into a continuous map of spatial dominance.

## Influence, Then Control

The construction is two-stage.

**Pitch influence** measures one player's reach. Each player is assigned a bivariate normal whose mean and covariance are adjusted for velocity and distance to the ball — so a sprinting player's influence stretches ahead of him, and players near the ball have tighter, more concentrated influence. Normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

**Pitch control** contests the two teams' influence at each location, squashed to a probability:

$$PC(p, t) = \sigma\!\left(\gamma\left(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\right)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ in the source, i.e. no team weighting and no shrinkage. Adapted from Fernández & Bornn's "Wide Open Spaces" (2018).

## Why It Matters

Pitch control is **infrastructure** rather than an end in itself. In the [[expected-value-possession-framework|EPV framework]] it appears as an input feature to the ball-drive and action-selection models, where it serves as the quantitative definition of *pressure*: an action is classified as under pressure when the ball carrier's pitch control falls below 0.4.

That definition is what makes the framework's risk analysis possible. Actions under pressure show a wider EPV-added distribution with more negative mass — more likely to lose value, but more upside when the press is beaten. Without a continuous control surface, "under pressure" would remain a judgement call.

More broadly, pitch control is one of the few tools in this vault that values **space rather than events**. Every event-data framework is blind to a player who creates a passing lane by dragging a marker away; a control surface sees it directly. Compare [[off-ball-value]], which asks the related but distinct question of what a location is worth if the ball arrives there.

## Assumptions Worth Knowing

- **Gaussian reach.** A normal distribution is a smooth stand-in for what is really a physical reachability problem — how fast can this player, at this velocity, get to that point. Physics-based alternatives (Spearman et al., 2017) model the arrival-time contest directly rather than via a density.
- **Influence is additive.** Two defenders covering a zone contribute independently, which overstates control where their coverage overlaps.
- **Parameters are set to 1, not fitted.** $\lambda_1$, $\lambda_2$ and $\gamma$ are free and left at defaults, so nothing has been tuned to make control predict anything in particular.
- **Control is not possession.** The surface says who *would* win the ball there, which is counterfactual for every location except the one the ball occupies — and is validated only through downstream model performance, never directly.

## See Also

- [[probability-surface]] · [[off-ball-value]] · [[optical-tracking-data]]
- [[dynamic-pressure-lines]] · [[tactical-analysis]]
- [[expected-possession-value]] · [[soccermap]]
- [[javier-fernandez]] · [[luke-bornn]]
- [[expected-value-possession-framework|Source Summary]]
