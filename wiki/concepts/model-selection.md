---
title: "Model Selection"
type: concept
tags: [model-selection, statistics, machine-learning, mixture-model, clustering, evaluation, identifiability, regularization]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/expected_value_possession_framework.md, raw/papers/beyond_expected_goals.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 45%
  generated: 10%
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

Used by [[football-event-sequences-point-process-mixture|Amezouwui et al.]] to choose the number of possession clusters — one of the few genuinely principled complexity choices in this vault.

**Held-out validation.** Fit on one partition, evaluate on another. Assumption-free and the default in machine learning, but it needs data to spare, and it silently assumes the partitions are exchangeable — an assumption [[football-defence-evaluation-vdep|VDEP]] strains by using five one-week blocks as folds.

## The Vault's Recurring Failure

**Five frameworks, five asserted free parameters, and no sensitivity analysis reported by any of them.**^[generated: the enumeration and the absence claim are both constructed here. rests-on: absence:no-sensitivity-analysis-in-held-sources — ⚠️ expires if a source is acquired that reports one, or if a held source is re-read and found to contain one]

| Parameter | Framework | Justification given |
|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | "Stylistic preference" |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Mean possession duration |
| $k = 5$ events | [[vdep]] | Domain intuition |
| $C \approx 3.9$ | [[vdep]] | Event frequency ratio |
| 4 s | [[c-obso]] | Prediction accuracy trade-off |

They are **not all the same kind of parameter**, which lumping them together obscures: horizons ($\epsilon$, $k$, 4 s) are likely self-limiting, since most credit falls near the event regardless. $\gamma$ is a *shape* parameter spanning nearly two orders of magnitude across its author's own proposed range. $C$ is a *trade-off weight* derived from a frequency ratio, which encodes how often each event happens and says nothing about how much each matters. See [[free-parameters-load-bearing]].

**The unused alternative:** fit the parameter by maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]] rather than asserting it. That treats the free parameter as what it is — a model-selection problem — and both criteria are already computed for other purposes.

## The Exception, and Why

[[beyond-expected-goals|Spearman]] fits all six of his parameters by MAP with stated priors, on five training matches.

> ### `physical-units-admit-priors`
>
> **Parameters with physical units admit priors from previous measurement; parameters expressing preference do not.**
>
> ^[generated: Spearman states where his priors come from but does not offer this as a rationale, and no source draws the contrast. Also on [[theory-based-modelling]] and the synthesis. rests-on: source:spearman-map-priors — a *single* source's parameter table, which makes this the most fragile of the vault's multi-page generated claims by dependency alone]

Where a parameter means "temporal uncertainty on intercept time, in seconds", a prior can be inherited from a 2017 fit. Where it means "stylistic preference for vertical attacking", it cannot.

If right, this is an argument for [[theory-based-modelling|physical modelling]] that goes beyond taste.

**What would falsify it.** A physically-parameterised model whose authors still assert values without priors — which would show the availability of priors is not what drives the practice. One source is thin evidence for a general claim about modelling culture.

Contrast the cases where selection *is* done well: [[nmstpp]]'s ablations on factorisation order and zone discretisation, and Fernández et al.'s per-component [[probability-calibration|calibration]] checks. Both change conclusions; both are cheap.

## When Selection Is Not Enough

**Non-[[identifiability]].** If parameters are not uniquely determined by the distribution, comparing fits is meaningless before the model is made identifiable.

**No ground truth.** Where the quantity of interest is unobservable — action value, defensive contribution — likelihood measures fit to a *proxy*, not to the target. This is why the vault leans on [[split-half-reliability|reliability]], [[predictive-validity]] and external criteria instead. See [[rare-event-proxy-targets]].

## See Also

- [[free-parameters-load-bearing]] — the open question on the five parameters
- [[mixture-model]] · [[clustering]] · [[identifiability]] · [[regularization]] · [[theory-based-modelling]]
- [[split-half-reliability]] · [[predictive-validity]] · [[probability-calibration]] · [[rare-event-proxy-targets]]
- [[temporal-discounting]] · [[vdep]] · [[c-obso]] · [[nmstpp]] · [[obso]]
- [[football-event-sequences-point-process-mixture|Possession Clustering Summary]] · [[beyond-expected-goals|Spearman Summary]]
