---
title: "Player Ratings as Time Series"
type: concept
tags: [sports-analytics, player-evaluation, time-series, smoothing, volatility, action-valuation, recruitment, evaluation]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.8
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Player Ratings as Time Series

Almost every [[action-valuation]] framework ends by collapsing a season of actions into **one number per player** — [[vaep|VAEP]] per 90, [[expected-threat|xT]] per 90, goals per 90. This page is about what that final aggregation throws away, and what becomes available if you refuse to do it.

## What a Single Number Cannot Say

[[football-performance-time-series|Mendes-Neves et al.]] frame the problem directly: performance is volatile, not stationary. A player's season average cannot distinguish between:

- a player improving steadily and one declining steadily to the same mean;
- a genuine level and a fortunate three-month run;
- a consistent contributor and one alternating brilliance with anonymity;
- a player at their peak and one who has not reached it.

Every one of these distinctions matters for [[recruitment|recruitment]], and all of them survive in a time series and die in an average.

## Construction

The recipe is model-agnostic — it takes any per-action value and produces a series.

1. **Aggregate per player per game.** Sum action values within each match.
2. **Normalise by exposure.** Value per minute played, not per game, so substitutes are comparable.
3. **Filter for sample adequacy.** Drop games below a minutes threshold (60 in the source); drop goalkeepers, whose value profile is incomparable.
4. **Choose an index.** See below.
5. **Smooth.** Per-game values are far too noisy to read directly.

Series can be built for *all* actions or restricted to a single action type — passes, dribbles, shots — which is where the approach earns its keep.

## The Index Choice

Two options, with a real trade-off:

| Index | Advantage | Cost |
|---|---|---|
| **Cumulative games played** | Regular spacing; immune to injuries and season breaks | Carries no information about absence; a rating never ages |
| **Calendar date** | Encodes injuries, benchings, transfers out of the league | Irregular index; missing-data handling required |

The source chooses game count for simplicity and then names it as a limitation: **with no time decay, a player retains their rating indefinitely after they stop playing.** A player who has not appeared in three years still carries their last 40-game average.

This is a genuine open problem rather than an oversight. The calendar index would fix rating persistence but introduces the question of what a player's value *is* during a long injury — undefined, decaying, or held constant.

## Short-Term and Long-Term Windows

Two [[smoothing|moving averages]] over the same series, serving different purposes:

| Metric | Window | Minimum | Interpretation |
|---|---|---|---|
| **Short-term** | 10 games | 5 | Recent *form* |
| **Long-term** | 40 games | 20 | Underlying *quality* |

Windows were set by trial and error on the training set, targeting the standard trade-off: capture a change in true level as fast as possible without overreacting to one match.

**Simple moving averages beat exponential ones here.** The source tested EMAs and rejected them as less robust to outliers — a sensible result given that football ratings contain rare, enormous, genuinely anomalous values (a hat-trick, a red card in the third minute). An EMA's recency weighting amplifies exactly those points.

The pair is more useful than either alone. The **gap** between short-term and long-term is a form indicator: persistently above means a player is outperforming their established level, and the sign of the gap flags improvement or decline before the long-term average has moved.

## What This Unlocks

Three derived quantities exist only in the time-series view:

- **[[performance-volatility]]** — consistency measured as deviation from a player's own trend, rather than variance across players.
- **[[player-development-curve]]** — the shape of performance against age, and where an individual sits on it.
- **Style evolution** — per-action-type series reveal *how* a player's value generation changes. The source's example: Messi's dribble value and pass value moved in opposite directions across a decade as Barcelona's midfield aged, a shift no season average could represent.

## Relation to Reliability

There is an unresolved tension with [[split-half-reliability]] worth stating explicitly.

Split-half reliability asks whether a rating computed on one random half of a season matches the other half. It treats within-season variation as **noise to be averaged away**, and penalises metrics whose values move around.

Time-series treatment assumes some of that same variation is **signal** — real form, real development, real style change.

Both are defensible and they are not reconcilable by argument alone. The distinguishing question is empirical: does the short-term deviation from a player's long-term level *predict anything*? If short-term form forecasts next-match contribution beyond the long-term average, it is signal. If not, it is noise and the reliability critique wins.

> **Gap in the vault.** No source here tests this. The [[predictive-validity]] results in [[action-valuation-frameworks-compared]] evaluate metrics at the team-match level, not player form at the individual level, so they do not settle it.

## Limitations

- **Data-hungry.** A 40-game long-term window with a 20-game minimum excludes fringe players, most young players, and anyone recently transferred — arguably the exact population recruitment most needs to assess. The source is explicit that lowering the threshold yields unstable ratings rather than a solution.
- **Inherits its base metric's flaws.** A series built from VAEP carries VAEP's offensive bias and goal-driven variance at every point.
- **No context normalisation.** Trends confound player development with changes in team quality, tactical role, league strength, and teammate quality. A rising curve may be a rising player or a transfer to a better side.

## See Also

- [[performance-volatility]]
- [[player-development-curve]]
- [[intent-vs-outcome-valuation]]
- [[action-valuation]]
- [[vaep]]
- [[split-half-reliability]]
- [[predictive-validity]]
- [[action-valuation-frameworks-compared]]
- [[football-performance-time-series|Source Summary]]
