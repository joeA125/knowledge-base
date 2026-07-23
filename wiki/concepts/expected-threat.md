---
title: "Expected Threat (xT)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, markov-model, dynamic-programming, interpretability, event-stream-data]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Expected Threat (xT)

Expected Threat (xT; Karun Singh, 2019) is a possession-based [[markov-model|Markov model]] for [[action-valuation|valuing on-the-ball actions]] in soccer. It assigns each zone of the pitch a value representing how dangerous a team is when in possession there, then values an action by the change in zone value it produces.

## Core Idea

Overlay an $M \times N$ grid on the pitch. The game state is represented by **nothing but the zone containing the ball** — an extremely compressed representation. Each zone $z$ gets a value $xT(z)$: the probability of scoring within the next few actions, given possession at $z$.

An action moving the ball from zone $z$ to zone $z'$ is then valued:

$$V_{xT}(a_i) = xT(z') - xT(z)$$

## The Recursion

Zone values satisfy a Bellman-style equation:

$$xT(z) = \underbrace{s_z \cdot xG(z)}_{\text{shoot now}} + \underbrace{m_z \cdot \sum_{z'} T_{z \to z'} \cdot xT(z')}_{\text{move and try later}}$$

where:
- $s_z$ = probability of shooting from zone $z$
- $xG(z)$ = probability that a shot from $z$ scores ([[expected-goals]])
- $m_z$ = probability of moving the ball from $z$
- $T_{z \to z'}$ = probability of moving to zone $z'$

Solved by [[value-iteration]] from an all-zero initialisation. After iteration $i$, $xT(z)$ is the probability of scoring within the next $i$ actions. [[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] find a $16 \times 12$ grid converges in 6 iterations.

Note that [[expected-goals|xG]] appears *inside* the recursion — xT is built on top of a shot-quality model rather than replacing it.

## Strengths

**[[interpretability]].** The entire model is $M \cdot N$ numbers, one per zone, directly visualisable as a heatmap. Any valuation can be explained by pointing at two cells. This is the sharpest contrast with [[vaep]], which needs a [[gradient-boosting|gradient-boosted ensemble]] over dozens of interacting features.

**Robustness.** In split-half testing, xT ratings correlate at $\rho = 0.89$ across random halves of a season, against 0.25 for VAEP — see [[split-half-reliability]]. Players are highly consistent in where on the pitch they perform which action types, so a purely zonal metric inherits that stability.

**Simplicity.** Dynamic programming over a small state space; no feature engineering, no hyperparameter tuning, trivially fast.

## Limitations

**Only values ball progression.** Because the state *is* the zone, xT can only value actions that move the ball between zones — passes, dribbles, crosses. Tackles, interceptions, and take-ons that stay within a zone are invisible. Hence xT and its relatives are often called **ball-progression models**.

**Zone granularity discards detail.** Many forward dribbles inside the penalty box receive a value of exactly zero because they do not cross a cell boundary — even though near goal, a metre of displacement materially changes scoring odds.

**No risk modelling.** xT is possession-based: it estimates the chance of a goal *within the current possession* and stops at a turnover. So it cannot value how an action changes the chance of *conceding*. A risky square pass in midfield that invites a counterattack is not penalised.

**No context.** No score difference, no time remaining, no information about the preceding actions. A shot after a through ball and a shot after beating two defenders are valued identically if they come from the same zone.

**No credit for finishing.** Shots are not valued directly, so a clinical finisher gains nothing. Sergio Agüero ranks 19th by VAEP but 109th by xT for exactly this reason.

## Position Among Valuation Frameworks

xT is the canonical example of the *possession-based* style in the [[action-valuation]] taxonomy. It is confusingly also called an "expected possession value" approach in the soccer literature — a name collision with basketball's [[expected-possession-value|EPV]] (Cervone et al.), which is a different and much heavier construction.

## See Also

- [[action-valuation]]
- [[vaep]]
- [[expected-goals]]
- [[value-iteration]]
- [[split-half-reliability]]
- [[markov-game]]
- [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]]
