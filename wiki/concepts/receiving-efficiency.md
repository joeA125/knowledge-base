---
title: "Receiving and Interception Efficiency"
type: concept
tags: [player-evaluation, expected-versus-actual, pass-modelling, sports-analytics, off-ball, evaluation, statistics, defensive-valuation]
sources: [raw/papers/physics_based_pass_probabilities.md]
confidence: 0.85
provenance:
  extracted: 75%
  inferred: 20%
  generated: 5%
  imported: 0%
  ambiguous: 0%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Receiving and Interception Efficiency

Rating a player by **how many passes he received or intercepted, against how many a model says he should have.**

$$\text{efficiency} = \frac{\text{actual receptions}}{\sum_i p_i}$$

where each $p_i$ comes from [[pass-probability-model|a per-pass reception model]] and the expectation follows from the [[poisson-binomial]]. Above 1 means over-performing the model; below means under-performing.

Introduced by [[physics-based-pass-probabilities|Spearman et al. (2017)]] as the first application of their pass model.

## What It Fixes

Traditional measures are **reception percentage** and **interception count**. Neither accounts for difficulty.

A defender who intercepts ten easy loose balls and one who intercepts four passes he had almost no chance of reaching score 10 and 4. Efficiency scores the second higher, because the model already knew the first ten were likely.

That is the same move [[expected-goals|xG]] makes for shooting, applied to receiving. The framework is general — see [[poisson-binomial]] — but this is the vault's only instance applied to *receiving* rather than scoring.

## The Position Findings

Computed across 38 matches, split by whether the pass originated from a teammate (receiving) or an opponent (interception):

| Position | Total | Receiving | Interception |
|---|---|---|---|
| Defender | 1.06 | 1.04 | **1.13** |
| Midfielder | 1.03 | 1.23 | 0.57 |
| Forward | 0.91 | **1.36** | 0.23 |
| Goalkeeper | 0.65 | 0.80 | 0.48 |

**Forwards over-receive by 36% and barely intercept.** Defenders lead interception. The total column conceals this almost completely — defenders and midfielders look near-identical at 1.06 and 1.03, and the split shows they are doing entirely different things.

That is the useful methodological point: **a composite efficiency is close to uninformative; the decomposition is where the signal is.**

## The Confound the Authors Name

Team strategy affects aggression, so over- or under-performance cannot be cleanly attributed to skill.

Their example: a striker may have a chance to disturb a pass between opposing defenders, but team instruction says hold the defensive shape. He declines an interception he could have made, and the metric records a failure.

This is the [[intent-vs-outcome-valuation|intent/outcome problem]] arriving from a third direction. It is also the same objection [[policy-modelling]] raises about behaviour policies generally — **observed behaviour encodes instruction as well as ability, and nothing here separates them.**

The forward's 0.23 interception efficiency is therefore ambiguous between "forwards cannot intercept" and "forwards are told not to try", and the data cannot distinguish them.

## Team-Level Validation

Aggregated per team over 38 matches, total reception efficiency correlates **0.64 with shots** and **0.70 with passes in the attacking third**. Pass value, the companion metric, reaches 0.63 and 0.83.

Concurrent correlation rather than [[predictive-validity|prediction]], and on 20 teams — suggestive rather than established. But it does show the metric tracks something team-level rather than being pure noise.

## Limitations

- **Inherits the pass model's errors**, including its systematic 11-point underestimate of completion rate. If the model is uniformly pessimistic, every player's efficiency is inflated.
- **The strategy confound is unresolved**, as above.
- **Independence is assumed** in the expectation — see [[poisson-binomial]]. Correlated events inflate true variance, so apparent over-performance is less significant than it looks.
- **No [[split-half-reliability|reliability]] figure**, so whether a player's efficiency replicates across halves of a season is unknown.
- **One team's matches**, 2015-16 Premier League.

## Why It Is Worth Keeping

It is the vault's clearest instance of **evaluating a player against a model's expectation rather than against a raw count**, applied outside shooting.

The pattern generalises to any action with a modelled success probability: [[symmetrical-duel-valuation|duels]], tackles, aerial contests. That nobody has extended it that way is a gap — [[duel-skill-rating]] rates duel ability by [[glicko-rating-system|Glicko]] paired comparison instead, which needs no difficulty model but also cannot say a duel was unusually hard.

## See Also

- [[pass-probability-model]] · [[poisson-binomial]] · [[expected-goals]] · [[action-valuation]]
- [[intent-vs-outcome-valuation]] · [[policy-modelling]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[predictive-validity]] · [[split-half-reliability]] · [[off-ball-value]] · [[recruitment]] · [[william-spearman]]
- [[physics-based-pass-probabilities|Source Summary]]
