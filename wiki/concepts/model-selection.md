---
title: "Model Selection"
type: concept
tags: [model-selection, statistics, machine-learning, mixture-model, clustering, evaluation, identifiability, regularization]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/expected_value_possession_framework.md, raw/papers/beyond_expected_goals.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 42%
  generated: 8%
  imported: 5%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Model Selection

Choosing among candidate models — how many components, how much history, which features — when a richer model always fits the training data better.

## Two Routes

**Penalised likelihood.** Add a complexity term to the fit:^[imported: standard statistical background, not from any held source]

$$\text{BIC} = -2\log \hat{L} + k \log n, \qquad \text{AIC} = -2\log \hat{L} + 2k$$

BIC penalises harder as $n$ grows and is consistent for the true model if it is in the candidate set; AIC targets predictive performance and tends to select larger models. Requires a likelihood, so it is available to [[mixture-model|model-based]] methods and not to heuristic ones.

Used by [[football-event-sequences-point-process-mixture|Amezouwui et al.]] to choose the number of possession clusters.

**Held-out validation.** Fit on one partition, evaluate on another. Assumption-free and the default in machine learning, but it needs data to spare, and it silently assumes the partitions are exchangeable — an assumption [[football-defence-evaluation-vdep|VDEP]] strains by using five one-week blocks as folds.

## The Vault's Recurring Failure

> **Superseded, 2026-07-27.** This section previously read *"five frameworks, five asserted free parameters, zero sensitivity analyses."* The absence half is no longer accurate. [[generalized-vdep-euro-location-analysis|Umemoto, Tsutsui & Fujii (2022)]] report a systematic sweep over the number of observed players with F1 at each value, and replace [[vdep|VDEP's]] $C$ on principle rather than asserting a new value. The claim now stands in weakened form below.

**Five frameworks assert a free parameter without justifying its value:**

| Parameter | Framework | Justification given | Status |
|---|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | "Stylistic preference" | Unexamined |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Mean possession duration | Unexamined |
| $k = 5$ events | [[vdep]] | Domain intuition | Unexamined; inherited unchanged by [[gvdep]] |
| $C \approx 3.9$ | [[vdep]] | Event frequency ratio | **Superseded** — [[gvdep]] replaces it with VAEP-derived score-scaled weights |
| 4 s | [[c-obso]] | Prediction accuracy trade-off | Unexamined |

**Sensitivity analysis is rare rather than absent.**^[generated: the enumeration and the weakened absence claim. rests-on: absence:no-sensitivity-analysis-on-horizon-parameters, source:gvdep-n-nearest-sweep — ⚠️ the surviving absence is now narrow: no source sweeps a *horizon or weighting* parameter. Re-check on every ingest.] GVDEP sweeps $n\_nearest$ from 0 to 11 and reports F1 at each — a genuine sensitivity analysis, just on **which inputs to include** rather than on a horizon or weighting parameter. So the practice exists in this literature; it has simply never been applied to the five parameters above.

That distinction matters, because it removes the excuse. A group that sweeps one parameter could sweep another.

They are also **not all the same kind of parameter**: horizons ($\epsilon$, $k$, 4 s) are likely self-limiting, since most credit falls near the event regardless. $\gamma$ is a *shape* parameter spanning nearly two orders of magnitude across its author's own proposed range. $C$ was a *trade-off weight* from a frequency ratio — and is the one that has now been fixed. See [[free-parameters-load-bearing]].

**The unused alternative** for the remaining four: fit by maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]] rather than asserting. GVDEP demonstrates a second route — derive the parameter from an existing model on a meaningful scale, as it does by weighting with VAEP.

## The Exception, and Why

[[beyond-expected-goals|Spearman]] fits all six of his parameters by MAP with stated priors, on five training matches.

> ### `physical-units-admit-priors`
>
> **Parameters with physical units admit priors from previous measurement; parameters expressing preference do not.**
>
> ^[generated: Spearman states where his priors come from but does not offer this as a rationale, and no source draws the contrast. Also on [[theory-based-modelling]] and the synthesis. rests-on: source:spearman-map-priors — a *single* source's parameter table, which makes this the most fragile of the vault's multi-page generated claims by dependency alone]

Where a parameter means "temporal uncertainty on intercept time, in seconds", a prior can be inherited from a 2017 fit. Where it means "stylistic preference for vertical attacking", it cannot.

**What would falsify it.** A physically-parameterised model whose authors still assert values without priors. Note GVDEP is a partial counter-example in the other direction: its weights have *score* units rather than physical ones, and it derives them from data anyway — suggesting interpretable units of any kind, not physical ones specifically, may be what enables principled estimation.

## When Selection Is Not Enough

**Non-[[identifiability]].** If parameters are not uniquely determined by the distribution, comparing fits is meaningless.

**No ground truth.** Where the quantity of interest is unobservable, likelihood measures fit to a *proxy*. This is why the vault leans on [[split-half-reliability|reliability]], [[predictive-validity]] and external criteria. See [[rare-event-proxy-targets]].

## See Also

- [[free-parameters-load-bearing]] — the open question on the remaining four
- [[gvdep]] · [[vdep]] · [[mixture-model]] · [[clustering]] · [[identifiability]] · [[theory-based-modelling]]
- [[split-half-reliability]] · [[predictive-validity]] · [[probability-calibration]] · [[rare-event-proxy-targets]]
- [[temporal-discounting]] · [[c-obso]] · [[nmstpp]] · [[obso]]
- [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[beyond-expected-goals|Spearman Summary]]
