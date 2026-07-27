---
title: "Smoothing"
type: concept
tags: [statistics, smoothing, time-series, sports-analytics, volatility, approximation]
sources: [raw/papers/football-performance-time-series.md]
confidence: 0.75
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Smoothing

Smoothing recovers an underlying trend from a noisy series by averaging over a neighbourhood of each point. It trades **responsiveness** against **stability**: a wider window gives a cleaner signal that reacts later to genuine change.

## The Core Trade-off

Every smoother is parameterised by something controlling how much history it pools — window length, decay rate, bandwidth. That parameter sets where the estimate sits between two failure modes:

| Too little smoothing | Too much smoothing |
|---|---|
| Trend is buried in noise | Real changes detected late |
| Overreacts to single observations | Genuine shifts get averaged away |

There is no universally correct setting, because it depends entirely on the ratio of signal to noise in the underlying process and on how costly a delayed detection is.

## Moving Average vs Exponential Moving Average

The two most common choices behave quite differently in the presence of outliers.

**Simple moving average (SMA).** Every observation in the window gets equal weight; observations outside get zero. An outlier contributes $1/n$ of the estimate for exactly $n$ periods, then drops out completely.

**Exponential moving average (EMA).** Weights decay geometrically with age, so recent observations dominate. An outlier has a *large* immediate effect that decays but never fully vanishes.

EMAs are usually preferred in finance and forecasting for their responsiveness and constant memory. But [[football-performance-time-series|Mendes-Neves et al.]] tested both on player rating series and **rejected EMAs as less robust to outliers**.

This is the right call for the domain rather than a general result. Football rating series contain rare, enormous, genuinely anomalous values — a hat-trick, a red card in the third minute, a freak deflection. The EMA's recency weighting amplifies precisely those points, so a single extraordinary match would dominate a player's apparent form for weeks. The SMA caps any single match's influence at $1/n$ and evicts it cleanly.

**The general principle:** where the noise distribution is heavy-tailed, uniform weighting is more robust than recency weighting. Where noise is well-behaved and responsiveness matters, EMAs win.

## Multiple Windows as a Signal

Two smoothers of different widths over the same series carry more information than either alone. Their **difference** is a trend indicator: when the short window sits persistently above the long one, the underlying level is rising faster than the long window has yet absorbed.

[[player-rating-time-series|Mendes-Neves et al. use exactly this]] — a 10-game short-term average against a 40-game long-term average, read as *form* against *quality*. The same construction underlies moving-average crossover rules in finance and is the basis of the deviation terms in [[performance-volatility|volatility metrics]].

## Minimum-Observation Thresholds

A window near the start of a series, or spanning a gap, contains fewer points than nominal. Estimates there are noisier but look identical to well-supported ones.

The standard guard is a minimum count — the source requires 5 of 10 games for the short window and 20 of 40 for the long, emitting nothing otherwise. This is a small detail with a large consequence: it means the metric is **undefined** for players early in a series, which is itself a form of [[selection-bias|selection]] on who can be evaluated at all.

## Choice of Index

Smoothing presupposes a notion of "nearby". For irregularly-spaced observations that choice is substantive: a 10-*game* window and a 10-*week* window differ whenever a player is injured, rotated, or between clubs. See [[player-rating-time-series]] for the trade-off as it arises in practice.

## Related Uses in This Vault

- [[player-development-curve]] — the age curve is smoothed after bias correction and before normalisation.
- [[label-smoothing]] — unrelated despite the name; that softens target *distributions* for regularisation rather than denoising a series.
- [[gaussian-process]] — a principled probabilistic alternative that yields uncertainty bands rather than a point estimate, at considerably higher cost.

## Limitations

- **Endpoint bias.** Centred windows are undefined at the ends of a series; trailing windows lag by roughly half the window. Since the most recent estimate is usually the one being acted on, this matters more than it appears.
- **Assumes local stationarity.** Smoothing over a genuine discontinuity — a position change, a serious injury — blends two different regimes into a meaningless average.
- **Hides the variance you may want.** The residual discarded by smoothing is exactly the quantity [[performance-volatility|volatility analysis]] is interested in. Smoothing and volatility are complementary readings of one decomposition.

## See Also

- [[player-rating-time-series]]
- [[performance-volatility]]
- [[player-development-curve]]
- [[gaussian-process]]
- [[selection-bias]]
- [[football-performance-time-series|Source Summary]]
