---
title: "Split-Half Reliability"
type: concept
tags: [statistics, evaluation, reliability, player-evaluation, sports-analytics, cognitive-science]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 32%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Split-Half Reliability

Split-half reliability measures whether a metric produces consistent results when computed on two disjoint halves of the same data. Split the observations randomly in two, compute the measure separately on each half, and correlate the results across subjects. High correlation means the metric is capturing a stable underlying property; low correlation means it is largely capturing noise.

It originates in psychometrics as a test of internal consistency, but applies to any repeated measurement.

## Reliability Is Not Validity

A metric can be highly reliable and still measure the wrong thing — a broken thermometer that always reads 20°C is perfectly reliable and useless. Reliability is a *necessary* but not sufficient condition: an unreliable metric cannot be valid, since it does not measure anything stably, but a reliable one may still be measuring something you do not care about.

This distinction is why the finding below is suggestive rather than decisive.

## The xT vs VAEP Result

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] split a Premier League season into two random disjoint subsets, computed each player's rating on each, and correlated:

| Model | Pearson $\rho$ |
|---|---|
| [[expected-threat\|xT]] | **0.89** |
| [[vaep]] (all actions) | **0.25** |
| VAEP (offensive value, ball-progressing actions only) | 0.59 |

The gap is dramatic. A VAEP rating computed on one half of a season tells you remarkably little about the same player's VAEP rating on the other half.

## Why the Gap Exists

The authors identify two causes:

**Goals dominate VAEP ratings.** VAEP assigns high values to goals, and goals are rare and noisy. For a defender, a difference of three goals across a sample can double or halve their rating. xT gives no credit for shooting at all, so it is immune to this.

**Zonal behaviour is stable; outcomes are not.** Players are highly consistent in which action types they perform in which pitch locations (Decroos & Davis, 2019b). A metric defined purely over zonal transitions therefore inherits that stability. A metric that depends on *what happened next* inherits the variance of what happened next.

## The Controlled Comparison

The most informative row is the third. Restricting VAEP to xT's action set (passes, dribbles, crosses) and dropping defensive value raises reliability from 0.25 to 0.59 — but still well short of 0.89.

So the gap is **not merely a matter of scope**. Even valuing the same actions on the same offensive dimension, VAEP's richer contextual state representation introduces variance that xT's coarse zonal representation does not. Context buys sensitivity at the cost of stability.

## The General Trade-off

This is a bias–variance trade-off dressed in applied clothing. xT's crude state representation is heavily biased — it cannot see risk, context, or finishing — but has low variance. VAEP's rich representation reduces that bias but pays in variance, which shows up directly as unstable player ratings.

Which is preferable depends on use. For season-long player recruitment, stability matters enormously and xT's reliability is a serious argument in its favour. For analysing individual passages of play, VAEP's context-sensitivity is what you actually want, and split-half reliability across a season is close to irrelevant.

## Relation to Other Evaluation Concepts

Reliability sits alongside [[probability-calibration|calibration]] as a property distinct from accuracy. A model can be accurate on average, well calibrated, and still yield unreliable per-subject aggregates if the quantity being aggregated is high-variance. VAEP is well calibrated (Brier 0.0138) yet unreliable at the player-rating level — the two coexist without contradiction.

## See Also

- [[expected-threat]]
- [[vaep]]
- [[action-valuation]]
- [[probability-calibration]]
- [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]]
