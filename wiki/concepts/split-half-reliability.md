---
title: "Split-Half Reliability"
type: concept
tags: [statistics, evaluation, reliability, predictive-validity, player-evaluation, sports-analytics, cognitive-science, volatility, selection-bias, time-series]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-performance-time-series.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 25%
  generated: 15%
  imported: 5%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Split-Half Reliability

Split-half reliability measures whether a metric produces consistent results when computed on two disjoint halves of the same data. Split the observations randomly in two, compute the measure separately on each half, and correlate the results across subjects. High correlation means the metric is capturing a stable underlying property; low correlation means it is largely capturing noise.

It originates in psychometrics as a test of internal consistency, but applies to any repeated measurement.

## Reliability Is Not Validity

A metric can be highly reliable and still measure the wrong thing — a broken thermometer that always reads 20°C is perfectly reliable and useless. Reliability is a *necessary* but not sufficient condition: an unreliable metric cannot be valid, since it does not measure anything stably, but a reliable one may still be measuring something you do not care about.

The complementary test is [[predictive-validity]] — does the metric forecast outcomes it should? The two can disagree, and neither alone settles whether a metric is good.

## The xT vs VAEP Result

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] split a Premier League season into two random disjoint subsets, computed each player's rating on each, and correlated:

| Model | Pearson $\rho$ |
|---|---|
| [[expected-threat\|xT]] | **0.89** |
| [[vaep]] (all actions) | **0.25** |
| VAEP (offensive value, ball-progressing actions only) | 0.59 |

A VAEP rating computed on one half of a season tells you remarkably little about the same player's VAEP rating on the other half.

## Why the Gap Exists

The authors identify two causes:

**Goals dominate VAEP ratings.** VAEP assigns high values to goals, and goals are rare and noisy. For a defender, a difference of three goals across a sample can double or halve their rating. xT gives no credit for shooting at all, so it is immune to this.

**Zonal behaviour is stable; outcomes are not.** Players are highly consistent in which action types they perform in which pitch locations (Decroos & Davis, 2019b). A metric defined purely over zonal transitions inherits that stability. A metric that depends on *what happened next* inherits the variance of what happened next.

## The Controlled Comparison

Restricting VAEP to xT's action set and dropping defensive value raises reliability from 0.25 to 0.59 — still well short of 0.89.

So the gap is **not merely a matter of scope**. Even valuing the same actions on the same offensive dimension, VAEP's richer contextual state representation introduces variance that xT's coarse zonal representation does not. Context buys sensitivity at the cost of stability.

## The General Trade-off

This is a bias–variance trade-off dressed in applied clothing.^[generated: the bias–variance framing is not stated by any held source; it is a gloss applied here] xT's crude state representation is heavily biased — it cannot see risk, context, or finishing — but has low variance. VAEP's rich representation reduces that bias and pays in variance, which shows up directly as unstable player ratings.

Which is preferable depends on use. For season-long recruitment, stability matters and xT's reliability is a serious argument in its favour. For analysing individual passages of play, VAEP's context-sensitivity is what you actually want, and split-half reliability across a season is close to irrelevant.

## Is the Instability in the Metric or the Player?

[[football-performance-time-series|Mendes-Neves et al.]] measure a superficially similar quantity — [[performance-volatility|deviation of a player's per-game rating from their own long-term level]] — but treat it as a **property of the player**, a real and decision-relevant fact about consistency, rather than as measurement noise.

This is not a difference of opinion. Written out, the two framings are reading the same variance component.^[generated: neither source draws this equivalence; it is constructed here from the two together]

### The decomposition

Model a player's per-game rating as a true level plus a deviation:

$$r_{ig} = \theta_i + \varepsilon_{ig}$$

Averaging $n/2$ games in each half and correlating across players gives the standard reliability identity:^[imported: standard psychometric result, not from any held source]

$$\rho = \frac{\sigma^2_\theta}{\sigma^2_\theta + 2\sigma^2_\varepsilon / n}$$

So $\rho = 0.25$ is **not a direct statement about VAEP's construction**. It says within-player game-to-game variance $\sigma^2_\varepsilon$ is large relative to between-player variance in true level $\sigma^2_\theta$.

Game-to-game volatility, $\sigma(r_G - r_{LT})$, estimates that same $\sigma_\varepsilon$ per player. **One variance, two names.** The reliability framing calls it noise because it assumes $\theta_i$ is constant; the time-series framing assumes $\theta_i(t)$ moves.

### Test 1 — the aggregation-ratio test

If deviations are i.i.d. noise around a fixed $\theta$, averaging over a 10-game window shrinks them by a computable factor. With the short window nested inside the long one (10 of the trailing 40 games):

$$\sigma(r_{ST} - r_{LT}) = \sqrt{\tfrac{120}{1600}}\,\sigma_\varepsilon = 0.274\,\sigma_\varepsilon
\qquad
\sigma(r_G - r_{LT}) = \sqrt{\tfrac{1560}{1600}}\,\sigma_\varepsilon = 0.987\,\sigma_\varepsilon$$

giving a predicted ratio under the pure-noise null of $\approx 0.28$.^[generated: this test is constructed here; no source proposes it]

(The downside-only truncation applies the same factor $\approx 0.584$ to both under normality, so the ratio is unchanged for the negative variants.)

**Interpretation.** A ratio materially above 0.28 means deviations at the 10–40 game scale exceed what i.i.d. noise permits — genuine slow-moving signal, and *form is real*. A ratio near 0.28 means apparent form is noise read as trend.

> ⚠️ **This test is currently blocked.** The league-median volatility figures it needs do not survive in the vault's copy of the source: the Fig. 4 table is a fabricated arithmetic ramp, not measurement. Any ratio computed from it is meaningless. See the Data Fidelity section of [[football-performance-time-series]]. Recovering the original PDF would likely settle it immediately.

### Test 2 — reliability of volatility itself

A second test, which does **not** depend on the corrupted table: compute the **split-half reliability of the volatility metric**. If $\varepsilon$ is exchangeable noise, a player's volatility in one half of a season predicts nothing about the other half. If volatility replicates, it is a stable trait and the noise reading is wrong.

This is runnable today from any season of match-level ratings, which the first test is not. See [[within-season-variation-noise-or-signal]], which develops both and sets out how to read them jointly — volatility can be a real trait without being *forecastable*, and the two tests separate those cases.

### Why this matters beyond the two pages

The [[recruitment]] recommendation currently rests on the noise reading: xT is preferred over VAEP for season-long decisions *because* it replicates better. If VAEP's low $\rho$ is substantially genuine player inconsistency, that recommendation is measuring the wrong thing.

It also means **reliability and volatility are not independent evidence** and should not be cited as two findings supporting one conclusion.

There is separately a [[selection-bias]] caveat on the reliability figures themselves. They are computed on players with enough minutes to rate, and minutes are awarded partly on performance — a restricted range attenuates correlations. Reported reliability is conservative for the full population, though comparisons *between* metrics on the same sample remain fair.

## Two Routes to a More Reliable Metric

The diagnosis — goals are rare, heavily weighted, and noisy — suggests two fixes, only one of which has been tested:

1. **Restrict scope** (what Van Roy et al. tested): drop finishing and defensive value. Recovers $\rho = 0.25 \to 0.59$.
2. **Withhold outcome information** ([[intent-vs-outcome-valuation|untested]]): value the *decision* rather than the result. An I-VAEP-style model never sees whether the shot went in, removing the dominant noise channel while retaining full action scope.

The second is attractive because it does not require narrowing what the metric covers. In the decomposition above it should raise $\rho$ by shrinking $\sigma^2_\varepsilon$ while leaving $\sigma^2_\theta$ largely intact, whereas restricting scope shrinks both.^[generated: the predicted direction follows from the decomposition above, which is itself constructed here] No source has measured it.

## Relation to Other Evaluation Concepts

Reliability sits alongside [[probability-calibration|calibration]] as a property distinct from accuracy. VAEP is well calibrated (Brier 0.0138) yet unreliable at the player-rating level — the two coexist without contradiction.

An open gap: no paper in this vault reports both split-half reliability *and* [[predictive-validity]] for the same metric. xT is the most reliable measured ($\rho = 0.89$); [[lpv]], [[hpus]] and [[obso]] are the most predictive. Whether the two align is untested.

## See Also

- [[within-season-variation-noise-or-signal]] — the open question, with both tests
- [[predictive-validity]] · [[performance-volatility]] · [[intent-vs-outcome-valuation]] · [[player-rating-time-series]]
- [[expected-threat]] · [[vaep]] · [[action-valuation]] · [[probability-calibration]] · [[selection-bias]] · [[recruitment]]
- [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
