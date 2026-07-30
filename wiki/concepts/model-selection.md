---
title: "Model Selection"
type: concept
tags: [model-selection, statistics, machine-learning, mixture-model, clustering, evaluation, identifiability, regularization]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Model Selection

Choosing among candidate models — how many components, how much history, which features — when a richer model always fits the training data better.

## Two Routes

**Penalised likelihood.** Add a complexity term to the fit:

$$\text{BIC} = -2\log \hat{L} + k \log n, \qquad \text{AIC} = -2\log \hat{L} + 2k$$

BIC penalises harder as $n$ grows and is consistent for the true model if it is in the candidate set; AIC targets predictive performance and tends to select larger models. Requires a likelihood, so it is available to [[mixture-model|model-based]] methods and not to heuristic ones.

Used by [[football-event-sequences-point-process-mixture|Amezouwui et al.]] to choose the number of possession clusters — one of the few genuinely principled complexity choices in this vault.

**Held-out validation.** Fit on one partition, evaluate on another. Assumption-free and the default in machine learning, but it needs enough data to spare, and it silently assumes the partitions are exchangeable — an assumption [[football-defence-evaluation-vdep|VDEP]] strains by using five one-week blocks as folds.

## The Vault's Recurring Failure

The striking pattern across the football literature held here is how rarely either route is used for the parameters that matter most.

**Five frameworks, five asserted free parameters, zero sensitivity analyses:**

| Parameter | Framework | Justification given |
|---|---|---|
| $\gamma = 0.95$ per second | [[temporal-discounting\|Shelopugin]] | "Stylistic preference" |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Mean possession duration |
| $k = 5$ events | [[vdep]] | Domain intuition |
| $C \approx 3.9$ | [[vdep]] | Event frequency ratio |
| 4 s horizon | [[c-obso]] | Prediction accuracy trade-off |

None is selected by BIC, held-out performance, or downstream metric quality. Each is defensible as a starting point; none is shown to be the right value, and $\gamma$ in particular spans nearly two orders of magnitude in credit weight across its proposed range ($0.9^{30} = 0.04$ against $0.99^{30} = 0.74$).

A defensible alternative exists and is unused: **fit the parameter by maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]]** rather than asserting it. That treats the free parameter as what it is — a model-selection problem — rather than as a modelling preference.

Contrast the cases where selection *is* done well: [[nmstpp]]'s ablations on factorisation order and zone discretisation, and [[expected-value-possession-framework|Fernández et al.]]'s per-component [[probability-calibration|calibration]] checks. Both change conclusions; both are cheap.

## When Selection Is Not Enough

Two conditions defeat these criteria entirely, and both appear here.

**Non-[[identifiability]].** If parameters are not uniquely determined by the distribution, comparing fits is meaningless before the model is made identifiable.

**No ground truth.** Where the quantity of interest is unobservable — action value, defensive contribution — likelihood measures fit to a *proxy*, not to the target. This is why the vault leans on [[split-half-reliability|reliability]], [[predictive-validity]] and external criteria instead. See [[rare-event-proxy-targets]] for what the proxy substitution costs.

## See Also

- [[mixture-model]] · [[clustering]] · [[identifiability]] · [[regularization]]
- [[split-half-reliability]] · [[predictive-validity]] · [[probability-calibration]]
- [[temporal-discounting]] · [[vdep]] · [[c-obso]] · [[nmstpp]]
- [[football-event-sequences-point-process-mixture|Possession Clustering Summary]]
