---
title: "Selection Bias"
type: concept
tags: [statistics, selection-bias, evaluation, machine-learning, sports-analytics, player-development, model-selection]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 55%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Selection Bias

Selection bias arises when the data you observe is not a representative sample of the population you want to describe, because **the process that decided what got into the dataset is correlated with the thing you are measuring**. The estimate is then biased no matter how much data you gather — more observations of a skewed sample converge on the wrong answer.

## The Core Structure

Every instance has the same shape:

1. A quantity of interest, $Y$.
2. A selection indicator $S$ determining whether a unit is observed.
3. **$S$ depends on $Y$** (or on something correlated with it).

Then $\mathbb{E}[Y \mid S = 1] \neq \mathbb{E}[Y]$, and the sample mean estimates the former while the analyst wants the latter.

The failure is invisible from inside the dataset. Nothing about the observed data looks wrong; the missing units are missing.

## The Age-Curve Case

The clearest instance in this vault is the [[player-development-curve]]. Performance ratings by age, computed naively from top-division data, show players holding their level into their late thirties.

The selection mechanism: **a 36-year-old plays in a top division only if he is exceptional.** Average 36-year-olds have retired or dropped down. The same applies at 17. So at the age extremes the sample is drawn from the upper tail, while the mid-twenties sample is close to the full population.

The measured curve therefore describes *who survived at each age*, not *how a player develops*. This is survivorship bias — the same structure as evaluating investment strategies on funds that still exist, or aircraft armour on planes that returned.

[[football-performance-time-series|Mendes-Neves et al.]] correct it by assuming a uniform true age distribution and discounting each age band by $1 - (\text{relative number of players at that age})$: sparser age bands are more selectively sampled, so their observed averages are discounted harder. The direction is right; the specific form is a heuristic rather than a derived estimator.

## Other Instances in This Vault

**Restriction of range in reliability estimates.** [[split-half-reliability]] is computed on players with enough minutes to rate. Since playing time is awarded partly on the basis of performance, the sample is truncated on a variable correlated with the metric. Correlations computed on a restricted range are attenuated relative to the full population, so reported reliability figures are conservative for the population of all players — though the comparison *between* metrics on the same sample remains fair.

**Minutes thresholds throughout.** Almost every [[player-rating-time-series|rating time series]] filters on games played (>60 minutes per game, ≥20 games for a long-term window). Every such filter selects on a performance-correlated variable. The consequence is that **the players hardest to evaluate are the ones systematically excluded** — young, fringe, and newly transferred players, exactly the recruitment targets where information is most valuable.

**Event data availability.** [[event-stream-data|Event]] and [[optical-tracking-data|tracking]] data exist for wealthy leagues and top divisions. Findings about action value are findings about elite football, and generalise to lower levels only by assumption.

## Distinguishing It From Related Problems

| Problem | What goes wrong |
|---|---|
| **Selection bias** | Which units are *observed* depends on the outcome |
| **Confounding** | A third variable causes both treatment and outcome |
| **Measurement bias** | The instrument systematically mis-measures |
| **Distribution shift** | Train and deploy populations differ, for any reason |

Selection bias is a *cause* of distribution shift, and the two are often conflated. The distinction matters because the remedies differ: shift can sometimes be handled by reweighting toward a known target distribution, whereas selection bias requires a model of the selection mechanism itself — and that mechanism is usually unobserved.

## Remedies

- **Model the selection process explicitly** (Heckman correction, truncated regression) — principled, but needs an instrument affecting selection but not outcome, rarely available.
- **Reweight toward a known target distribution** — requires knowing the target, which is the assumption doing the work in the PDC correction.
- **Find data on the unselected units** — the real fix where possible. For age curves this means lower divisions, academies, and released players.
- **Report the restriction honestly** and describe the estimand as "among players who met the threshold" rather than "among players".

The last is underrated. Most selection bias in applied sports analytics is not corrected but *misdescribed*: a result about elite retained players is stated as a result about players.

## Relation to Counterfactual Modelling

[[counterfactual-simulation]] attacks a related problem from the other end. Observational player value is contaminated by the context a player was *selected into* — a good player at a strong club accumulates value partly because of the club. Simulating a player in a different lineup asks what they would produce absent that selection, making it one of the few approaches in the vault engaging directly with the fact that observed football data is not randomly assigned.

## See Also

- [[player-development-curve]]
- [[player-rating-time-series]]
- [[split-half-reliability]]
- [[predictive-validity]]
- [[counterfactual-simulation]]
- [[event-stream-data]]
- [[football-performance-time-series|Source Summary]]
