---
title: "Predictive Validity"
type: concept
tags: [predictive-validity, evaluation, reliability, statistics, sports-analytics, player-evaluation, transfer-prediction, recruitment]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
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

## The Team-Level Result

[[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]], following Spearman (2018) and Davis et al. (2024), correlate each metric in one match against outcomes in a team's *subsequent* match:

| | poss-util | [[hpus\|HPUS]] | [[lpv\|LPV]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

The headline is not that LPV wins. It is that **HPUS and LPV both beat xG and goals at predicting the next match** — including at predicting *goals themselves*.

Goals are the worst predictor of future goals in the table ($\rho = 0.11$). This is the sports-analytics version of regression to the mean: a scoreline is a small, noisy sample of an underlying process, while a possession-value metric aggregates hundreds of actions and estimates that process more directly. Measuring the process beats measuring the outcome when the outcome is sparse.

## The Player-Level Result

The table above is a **team-match** result, and [[action-valuation-frameworks-compared|the synthesis]] has flagged that it does not license player-level conclusions. [[epv-control-duel-skills-football|Shelopugin]] supplies the vault's first player-level evidence.

The setup: predict a player's [[pass-carry-reward|PCR]] for the *following season*, against a persistence baseline that simply carries this season's value forward.

| Sample | Baseline RMSE | Model RMSE |
|---|---|---|
| All data (>100 min) | 0.053 | **0.033** |
| Same team, same league | 0.050 | **0.032** |
| Same team, new league | 0.051 | **0.031** |
| New team, same league | 0.055 | **0.034** |
| New team, new league | 0.061 | **0.037** |

Two readings, and the second matters more.

**The baseline degrades monotonically with movement.** Last season's number predicts next season's worst for players who change both club and league — exactly the population [[recruitment]] cares about. Persistence is a decent heuristic only for players who stay put.

**The model's advantage holds across all five strata.** It does not collapse under the distribution shift that breaks the baseline, which suggests the [[league-strength-rating|league and club strength features]] are carrying real information about context transfer rather than merely fitting the stay-put majority.

## A Weaker Test Than It Appears

This result should be read carefully, because it is not the same test as the team-level table.

Hirnschall & Bajons predict an **independent outcome** — a metric in one match against *xG and goals* in the next. Shelopugin predicts **the metric's own future value**. Those differ in what they can establish:

| | Predicts | Rules out |
|---|---|---|
| Team-level | An external outcome | A metric that stably measures the wrong thing |
| Player-level (PCR) | Itself, one season later | Only pure noise |

A metric that measured, say, a player's tactical role rather than their quality would be highly self-predictable across seasons and would score well here. Self-prediction establishes that a metric captures something **persistent**; it does not establish that the persistent thing is skill.

The author is candid about this — the paper states outright that there is no mathematical way to demonstrate EPV-based metrics track player ability, and proposes two unexecuted alternatives: expert review of the shortlists, and checking them against actual transfers to elite clubs on the assumption that the top of the market is efficient.

So the honest summary is that the vault now has **player-level forecast evidence but still no player-level construct validation**. The gap the synthesis flagged is narrowed rather than closed.

## Why This Is the Right Test Here

Predictive validity suits situations where the quantity of interest is a persistent latent property — team quality, player ability — observed only through noisy realisations.

It is also naturally resistant to a failure mode that catches concurrent correlation. A metric that simply recomputed goals would score perfectly on same-match correlation with goals and poorly on predicting future goals, since it would inherit all the noise. Requiring forecast performance forces the metric to capture signal rather than echo the outcome.

## Relation to Reliability

The two are complementary and can disagree. A metric could be highly reliable ([[split-half-reliability|split-half]] ρ near 1) yet forecast nothing, if it stably measures an irrelevant property. Conversely a noisy metric cannot forecast well, since its noise propagates — so **reliability is necessary but not sufficient for predictive validity**.

The vault's evidence is consistent with this: xT is the most reliable metric (ρ = 0.89), and HPUS and LPV are the most predictive. No single paper reports both for the same metric, which remains a gap in the literature rather than in these notes — and PCR is now the sharpest instance of it, since its forecastability is documented in detail while its reliability is not reported at all.

## Beyond Sport

The distinction is standard in psychometrics, where predictive validity means a test forecasts a criterion outcome — an aptitude test predicting job performance, for instance. It sits alongside construct validity (does it measure the intended concept?) and content validity (does it cover the domain?).

The PCR case is a clean illustration of why the psychometric tradition keeps these separate. Demonstrating that a test score predicts *next year's test score* is a reliability finding dressed as a validity one; predictive validity requires an external criterion.

Machine learning's held-out test set is a form of predictive validity, but a weaker one still: it tests generalisation to unseen samples from the same distribution, not forecasting of a genuinely future outcome.

## See Also

- [[split-half-reliability]] · [[selection-bias]]
- [[lpv]] · [[hpus]] · [[pass-carry-reward]]
- [[transfer-performance-prediction]] · [[recruitment]] · [[league-strength-rating]]
- [[action-valuation]] · [[expected-goals]]
- [[action-valuation-frameworks-compared]]
- [[understanding-football-possessions-path-signatures|Path Signatures Summary]]
- [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
