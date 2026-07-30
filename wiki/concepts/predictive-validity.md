---
title: "Predictive Validity"
type: concept
tags: [predictive-validity, evaluation, reliability, statistics, sports-analytics, player-evaluation, transfer-prediction, recruitment, off-ball]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/beyond_expected_goals.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 37%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Predictive Validity

Predictive validity asks whether a metric forecasts future outcomes it ought to forecast. It is one of the few available answers to a hard problem: **how do you validate a metric when there is no ground truth?**

## The Problem It Solves

Football possession-value metrics have no correct answer to compare against. When [[expected-threat|xT]] and [[vaep]] disagree about a through ball, nothing adjudicates — [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] state plainly that determining true action values is "very difficult, if not impossible."

Four strategies have emerged in this vault, answering different questions:

| Strategy | Question | Example |
|---|---|---|
| **Concurrent correlation** | Does it agree with known-good indicators *now*? | [[hpus]] vs same-season xG (0.92) |
| **[[split-half-reliability\|Reliability]]** | Is the measurement *stable*? | xT ρ=0.89 vs VAEP ρ=0.25 |
| **Predictive validity** | Does it forecast *future* outcomes? | See below |
| **External criterion** | Does it track something outside the pipeline? | [[c-obso]] vs salary (0.45) |

None is sufficient alone. Concurrent correlation risks rewarding redundancy. Reliability can be high for something measuring the wrong thing.

## The Strongest Result: Player-Level, External Outcome

[[beyond-expected-goals|Spearman (2018)]] correlates each player's match-$i$ value against their match-$(i{+}1)$ output across 53 matches:

| Predictor ↓ / Outcome → | Next-match OBSO | Next-match shots | **Next-match goals** |
|---|---|---|---|
| **[[obso\|OBSO]]** | 0.60 | 0.37 | **0.26** |
| Shots | ~ | 0.35 | 0.17 |
| Goals | ~ | ~ | 0.12 |

**A player's off-ball positioning predicts his next-match goals better than his shots or his goals do.**

This is the vault's best predictive-validity evidence, and it clears three bars the other results do not:

- **Player-level**, not team-level.
- **Against an independent outcome** (goals), not the metric's own future value.
- **Beating the outcome's own lagged value** — the strongest available form, since it shows the metric carries signal the outcome itself does not.

Spearman states this as the design objective outright: build **a leading indicator less stochastic than scoring itself.**

## The Team-Level Result

[[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]], following Spearman (2018) and Davis et al. (2024), correlate each metric in one match against outcomes in a team's *subsequent* match:

| | poss-util | [[hpus\|HPUS]] | [[lpv\|LPV]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

Same shape as Spearman's, one level up: **HPUS and LPV both beat xG and goals at predicting the next match** — including at predicting goals themselves.

**Goals are the worst predictor of future goals** in both tables ($\rho = 0.11$ and $0.12$, arrived at independently seven years apart). This is regression to the mean in its sports form: a scoreline is a small, noisy sample of an underlying process, while a possession-value metric aggregates hundreds of actions and estimates that process directly. **Measuring the process beats measuring the outcome when the outcome is sparse.**

## The Weaker Player-Level Result

[[epv-control-duel-skills-football|Shelopugin]] predicts a player's next-season [[pass-carry-reward|PCR]] against a persistence baseline:

| Sample | Baseline RMSE | Model RMSE |
|---|---|---|
| All data (>100 min) | 0.053 | **0.033** |
| Same team, same league | 0.050 | **0.032** |
| New team, new league | 0.061 | **0.037** |

Useful for what it shows about *movement*: persistence degrades monotonically as a player changes club and league, while the model's advantage holds — exactly the population [[recruitment]] cares about.

But it predicts **the metric's own future value**, not an independent outcome:

| | Predicts | Rules out |
|---|---|---|
| Spearman, Hirnschall & Bajons | An external outcome | A metric that stably measures the wrong thing |
| Shelopugin (PCR) | Itself, one season later | Only pure noise |

Self-prediction establishes that a metric captures something **persistent**; it does not establish that the persistent thing is skill. A metric tracking tactical role rather than quality would score just as well.

## Cross-Horizon Consistency

[[vdep]] adds a check nobody else reports, and it is nearly free. Its match-level and season-level correlations are similar (0.464, 0.397) while VAEP's diverge sharply (0.830 → 0.177).

A metric that tracks the match it measures but not the season is **reproducing the scoreline**; one whose correlations hold across horizons is measuring a team property. Worth adopting as standard.

## Why This Is the Right Test Here

Predictive validity suits situations where the quantity of interest is a persistent latent property observed only through noisy realisations.

It is also resistant to a failure mode that catches concurrent correlation. A metric that simply recomputed goals would score perfectly on same-match correlation with goals and poorly on predicting future goals, inheriting all the noise. Requiring forecast performance forces the metric to capture signal rather than echo the outcome.

## Relation to Reliability

Complementary, and they can disagree. A metric could be highly reliable yet forecast nothing, if it stably measures an irrelevant property. Conversely a noisy metric cannot forecast well, since its noise propagates — so **reliability is necessary but not sufficient for predictive validity.**

The vault's evidence is consistent: xT is the most reliable (ρ = 0.89), while OBSO, HPUS and LPV are the most predictive. No single source reports both for the same metric — a gap in the literature rather than in these notes, and PCR is the sharpest instance, with its forecastability documented in detail and its reliability unreported.

## Beyond Sport

Standard in psychometrics, where predictive validity means a test forecasts a criterion outcome — an aptitude test predicting job performance. It sits alongside construct validity (does it measure the intended concept?) and content validity (does it cover the domain?).

The PCR case illustrates why the tradition keeps these separate: demonstrating that a test score predicts *next year's test score* is a reliability finding dressed as a validity one. Machine learning's held-out test set is weaker still — generalisation to unseen samples from the same distribution, not forecasting of a genuinely future outcome.

## See Also

- [[obso]] · [[split-half-reliability]] · [[selection-bias]] · [[rare-event-proxy-targets]]
- [[hpus]] · [[lpv]] · [[pass-carry-reward]] · [[c-obso]] · [[vdep]]
- [[transfer-performance-prediction]] · [[recruitment]] · [[action-valuation]]
- [[action-valuation-frameworks-compared]]
- [[beyond-expected-goals|Spearman Summary]] · [[understanding-football-possessions-path-signatures|Path Signatures Summary]] · [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
