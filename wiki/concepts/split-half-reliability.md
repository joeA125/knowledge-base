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

Split-half reliability measures whether a metric produces consistent results when computed on two disjoint halves of the same data. Split the observations randomly, compute the measure on each half, and correlate across subjects. High correlation means the metric captures a stable underlying property; low correlation means it largely captures noise.

It originates in psychometrics as a test of internal consistency, but applies to any repeated measurement.

## Reliability Is Not Validity

A metric can be highly reliable and still measure the wrong thing — a broken thermometer that always reads 20°C is perfectly reliable and useless. Reliability is *necessary* but not sufficient.

The complementary test is [[predictive-validity]]. The two can disagree, and neither alone settles whether a metric is good.

## The xT vs VAEP Result

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] split a Premier League season into two random disjoint subsets, computed each player's rating on each, and correlated:

| Model | Pearson $\rho$ |
|---|---|
| [[expected-threat\|xT]] | **0.89** |
| [[vaep]] (all actions) | **0.25** |
| VAEP (offensive value, ball-progressing actions only) | 0.59 |

## Why the Gap Exists

**Goals dominate VAEP ratings.** Goals are rare and noisy; for a defender, three goals across a sample can double or halve a rating. xT gives no credit for shooting, so it is immune.

**Zonal behaviour is stable; outcomes are not.** Players are highly consistent in which action types they perform in which locations. A metric defined over zonal transitions inherits that stability; one depending on *what happened next* inherits the variance of what happened next.

## The Controlled Comparison

Restricting VAEP to xT's action set and dropping defensive value raises reliability from 0.25 to 0.59 — still well short of 0.89. So the gap is **not merely a matter of scope**: the richer contextual state representation itself introduces variance.

This is a bias–variance trade-off dressed in applied clothing.^[generated: the framing is a gloss applied here, not stated by any source. rests-on: source:vanroy-controlled-comparison] For season-long [[recruitment]], stability matters and xT's reliability is a serious argument in its favour. For analysing individual passages of play, VAEP's context-sensitivity is what you want.

## Is the Instability in the Metric or the Player?

[[performance-volatility|Mendes-Neves et al.]] measure a superficially similar quantity but treat it as a **property of the player** rather than measurement noise.

> ### `reliability-volatility-identity`
>
> **Split-half reliability and performance volatility measure the same variance component, with opposite interpretations.**
>
> ^[generated: neither source draws this equivalence; it is constructed here from the two together. Also on [[performance-volatility]]. rests-on: imported:psychometric-decomposition, source:vanroy-rho-values, source:mendes-neves-volatility-definitions — note the load-bearing premise is *imported*, so nothing in the vault can check it]

### The decomposition

Model a player's per-game rating as a true level plus a deviation:

$$r_{ig} = \theta_i + \varepsilon_{ig}$$

Averaging $n/2$ games in each half and correlating across players gives the standard reliability identity:^[imported: standard psychometric result, not from any held source]

$$\rho = \frac{\sigma^2_\theta}{\sigma^2_\theta + 2\sigma^2_\varepsilon / n}$$

So $\rho = 0.25$ is **not a direct statement about VAEP's construction**. It says within-player game-to-game variance $\sigma^2_\varepsilon$ is large relative to between-player variance in true level $\sigma^2_\theta$.

Game-to-game volatility, $\sigma(r_G - r_{LT})$, estimates that same $\sigma_\varepsilon$. **One variance, two names.** The reliability framing calls it noise because it assumes $\theta_i$ is constant; the time-series framing assumes $\theta_i(t)$ moves.

### Test 1 — the aggregation-ratio test

If deviations are i.i.d. noise around a fixed $\theta$, averaging over a 10-game window shrinks them by a computable factor. With the short window nested inside the long one:

$$\sigma(r_{ST} - r_{LT}) = 0.274\,\sigma_\varepsilon
\qquad
\sigma(r_G - r_{LT}) = 0.987\,\sigma_\varepsilon$$

giving a predicted ratio under the pure-noise null of $\approx 0.28$.^[generated: this test is constructed here; no source proposes it. rests-on: claim:reliability-volatility-identity — cascades if that identity is wrong]

A ratio materially above 0.28 means genuine slow-moving signal, and *form is real*. Near 0.28 means apparent form is noise read as trend.

> ⚠️ **Currently blocked.** The league-median volatility figures it needs do not survive in the vault's copy of the source: the Fig. 4 table is a fabricated arithmetic ramp, not measurement. See the Data Fidelity section of [[football-performance-time-series]].

### Test 2 — reliability of volatility itself

Does **not** depend on the corrupted table: compute the **split-half reliability of the volatility metric**. If $\varepsilon$ is exchangeable noise, a player's volatility in one half predicts nothing about the other. If volatility replicates, it is a stable trait and the noise reading is wrong.

Runnable today from any season of match-level ratings. See [[within-season-variation-noise-or-signal]], which develops both and sets out how to read them jointly.

### Why this matters beyond the two pages

The [[recruitment]] recommendation currently rests on the noise reading. If VAEP's low $\rho$ is substantially genuine player inconsistency, that recommendation is measuring the wrong thing.

It also means **reliability and volatility are not independent evidence** and should not be cited as two findings supporting one conclusion.

There is separately a [[selection-bias]] caveat on the reliability figures. They are computed on players with enough minutes to rate, and minutes are awarded partly on performance — a restricted range attenuates correlations. Comparisons *between* metrics on the same sample remain fair.

## Two Routes to a More Reliable Metric

1. **Restrict scope** (tested): drop finishing and defensive value. Recovers $\rho = 0.25 \to 0.59$.
2. **Withhold outcome information** ([[intent-vs-outcome-valuation|untested]]): value the *decision* rather than the result, removing the dominant noise channel while retaining full action scope.

The second should raise $\rho$ by shrinking $\sigma^2_\varepsilon$ while leaving $\sigma^2_\theta$ intact, whereas restricting scope shrinks both.^[generated: follows from the decomposition above. rests-on: claim:reliability-volatility-identity] No source has measured it.

## Relation to Other Evaluation Concepts

VAEP is well calibrated (Brier 0.0138) yet unreliable at the player-rating level — [[probability-calibration|calibration]] and reliability coexist without contradiction.

**No paper in this vault reports both split-half reliability and [[predictive-validity]] for the same metric.**^[generated: an absence claim. rests-on: absence:no-source-reports-both — ⚠️ re-check on every ingest] xT is the most reliable measured; [[lpv]], [[hpus]] and [[obso]] are the most predictive. Whether the two align is untested.

## See Also

- [[within-season-variation-noise-or-signal]] — the open question, with both tests
- [[predictive-validity]] · [[performance-volatility]] · [[intent-vs-outcome-valuation]] · [[player-rating-time-series]]
- [[expected-threat]] · [[vaep]] · [[action-valuation]] · [[probability-calibration]] · [[selection-bias]] · [[recruitment]]
- [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|Source Summary]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
