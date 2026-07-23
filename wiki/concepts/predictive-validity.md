---
title: "Predictive Validity"
type: concept
tags: [predictive-validity, evaluation, reliability, statistics, sports-analytics, player-evaluation]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 33%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Predictive Validity

Predictive validity asks whether a metric forecasts future outcomes it ought to forecast. It is one of the few available answers to a hard problem: **how do you validate a metric when there is no ground truth?**

## The Problem It Solves

Football possession-value metrics have no correct answer to compare against. When [[expected-threat|xT]] and [[vaep]] disagree about a through ball, nothing adjudicates — [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] state plainly that determining true action values is "very difficult, if not impossible."

Three strategies have emerged in this vault, and they answer different questions:

| Strategy | Question | Example |
|---|---|---|
| **Concurrent correlation** | Does it agree with known-good indicators *now*? | [[hpus]] vs same-season xG (0.92) |
| **[[split-half-reliability\|Reliability]]** | Is the measurement *stable*? | xT ρ=0.89 vs VAEP ρ=0.25 |
| **Predictive validity** | Does it forecast *future* outcomes? | [[lpv]] vs next-match xG (0.32) |

None is sufficient alone. Concurrent correlation risks rewarding a metric for being redundant with what it correlates against. Reliability can be high for something measuring the wrong thing. Predictive validity is the most demanding, but a metric could in principle forecast well while being uninterpretable.

## The Football Result

[[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]], following Spearman (2018) and Davis et al. (2024), correlate each metric in one match against outcomes in a team's *subsequent* match:

| | poss-util | [[hpus\|HPUS]] | [[lpv\|LPV]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

The headline is not that LPV wins. It is that **HPUS and LPV both beat xG and goals at predicting the next match** — including at predicting *goals themselves*.

Goals are the worst predictor of future goals in the table ($\rho = 0.11$). This is the sports-analytics version of regression to the mean: a scoreline is a small, noisy sample of an underlying process, while a possession-value metric aggregates hundreds of actions and estimates that process more directly. Measuring the process beats measuring the outcome when the outcome is sparse.

## Why This Is the Right Test Here

Predictive validity suits situations where the quantity of interest is a persistent latent property — team quality, player ability — observed only through noisy realisations.

It is also naturally resistant to a failure mode that catches concurrent correlation. A metric that simply recomputed goals would score perfectly on same-match correlation with goals and poorly on predicting future goals, since it would inherit all the noise. Requiring forecast performance forces the metric to capture signal rather than echo the outcome.

## Relation to Reliability

The two are complementary and can disagree. A metric could be highly reliable ([[split-half-reliability|split-half]] ρ near 1) yet forecast nothing, if it stably measures an irrelevant property. Conversely a noisy metric cannot forecast well, since its noise propagates — so **reliability is necessary but not sufficient for predictive validity**.

The vault's evidence is consistent with this: xT is the most reliable metric (ρ = 0.89), and HPUS and LPV are the most predictive. No single paper reports both for the same metric, which is a gap in the literature rather than in these notes.

## Beyond Sport

The distinction is standard in psychometrics, where predictive validity means a test forecasts a criterion outcome — an aptitude test predicting job performance, for instance. It sits alongside construct validity (does it measure the intended concept?) and content validity (does it cover the domain?).

Machine learning's held-out test set is a form of predictive validity, but a weaker one: it tests generalisation to unseen samples from the same distribution, not forecasting of a genuinely future outcome.

## See Also

- [[split-half-reliability]]
- [[lpv]]
- [[hpus]]
- [[action-valuation]]
- [[expected-goals]]
- [[action-valuation-frameworks-compared]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
