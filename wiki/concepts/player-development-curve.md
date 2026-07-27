---
title: "Player Development Curve (PDC)"
type: concept
tags: [sports-analytics, player-development, player-evaluation, time-series, selection-bias, statistics, recruitment, smoothing]
sources: [raw/papers/football-performance-time-series.md]
confidence: 0.75
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Player Development Curve (PDC)

A player development curve describes **expected performance as a function of age** — the average shape of a career. It answers where a player sits relative to their likely peak, which is the question underneath most transfer decisions and every academy investment.

## Construction

[[football-performance-time-series|Mendes-Neves et al.]] build it from [[player-rating-time-series|rating time series]]:

1. Restrict to players with **more than one season** of data.
2. For each player, compute median rating **grouped by age**.
3. **Normalise each player's curve by their own maximum**, so every player peaks at 1.0.
4. Take the **median across players** at each age → the unadjusted curve.
5. Apply the [[selection-bias|selection-bias correction]] (below).
6. Smooth, and normalise to $[0, 1]$.

Step 3 is what makes the curve about *shape* rather than *level*. Without it, the curve would be dominated by the fact that better players play more.

## The Bias That Makes It Hard

The unadjusted curve is badly wrong, and wrong in an instructive way: it shows performance staying roughly flat into a player's late thirties.

The reason is a textbook [[selection-bias|selection effect]]. **A 36-year-old is only in a top division if he is exceptional.** So is a 17-year-old. Average players of those ages exist — in academies, in lower divisions, retired — but not in the dataset. The observed sample at the age extremes is drawn from the upper tail of the true population, so the curve measures *who survived*, not *how players develop*.

This is survivorship bias in its cleanest form, and it is why naive age-curve analysis reliably overstates the longevity of careers.

### The correction

Assume the true population is **uniform across ages**. Then compute, for each age, the *relative amount of players* — the count at that age divided by the maximum count at any age — and multiply each age's value by:

$$1 - \text{relative amount of players}$$

The logic: an age band that is *underrepresented* is being sampled more selectively, so its observed average is inflated more and needs a larger discount.

> **This is a heuristic, not an estimator.** It has the right qualitative behaviour — it discounts sparse age bands harder — but the specific functional form is not derived from any model of the selection mechanism. A principled alternative would model the selection probability explicitly (a truncated-distribution or Heckman-style correction) and recover the untruncated mean. The paper offers no justification for $1 - r$ over any other decreasing function, and the *magnitude* of the correction is therefore arbitrary even if its direction is right.
>
> The result nevertheless agrees with independent work, which is meaningful evidence that the correction is not badly miscalibrated.

## The Result

**Peak performance falls between 25 and 27**, consistent with Dendir (2016), reached by different methods. The corrected curve rises steeply through the early twenties, peaks, and declines through the thirties — the shape practitioners assume, now recovered from action-valuation data rather than assumed.

## Applications

**Locating a player on the curve.** A 23-year-old at 0.8 of their own peak is a different asset from a 30-year-old at 0.8, even at identical current output. One is appreciating, the other depreciating.

**Finding late bloomers — the sharpest use.** Market value drops sharply once a player passes the nominal peak age, because pricing assumes the average curve applies to everyone. Players who actually peak in their mid-thirties are therefore systematically underpriced. The source names Tiago, Aritz Aduriz, and Joaquín as examples in the La Liga data.

This is an inefficiency argument: the mispricing exists precisely *because* the average curve is widely believed. A club willing to evaluate the individual curve rather than the population one can buy real production cheaply.

**Projection.** Combining a player's position on the curve with their current level gives a crude forecast of remaining career value — useful for contract length and amortisation decisions.

## Limitations

- **A population median is not an individual trajectory.** The curve describes an average shape; individual careers vary enormously in peak age and decline rate. Using it to project a specific player assumes the very homogeneity that the late-bloomer application depends on violating. The two uses are in tension.
- **Age confounds role.** Players often shift position as they age — a winger becoming a central midfielder — changing how [[action-valuation|action value]] accrues independently of ability. The curve reads this as development or decline.
- **Uniform-age assumption is false.** The true population of *professional* footballers is not uniform across ages; genuine attrition happens for reasons other than selection into the dataset. The correction over-adjusts to the extent that this is true.
- **Single source, single league.** Built on La Liga 2009–2019 only. The peak-age agreement with Dendir is the sole external check.
- **Inherits the base metric's bias.** Built on a VAEP variant, so it is a curve for *on-ball attacking contribution*, not for playing ability. Defenders and goalkeepers — the latter excluded outright — may have quite different true curves.

## See Also

- [[selection-bias]]
- [[player-rating-time-series]]
- [[performance-volatility]]
- [[action-valuation]]
- [[vaep]]
- [[football-performance-time-series|Source Summary]]
