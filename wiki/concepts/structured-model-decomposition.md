---
title: "Structured Model Decomposition"
type: concept
tags: [model-decomposition, machine-learning, interpretability, statistics, sports-analytics, action-valuation, calibration, evaluation]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Structured Model Decomposition

Estimating a single quantity by expanding it algebraically into subcomponents, fitting a separate model to each, and recombining. The alternative to training one model end-to-end on the target.

## The Construction

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] give the clearest worked example in this vault. The target is $\mathbb{E}[G|T_t]$ — which team scores next. Applying the law of total expectation over the available actions:

$$EPV_t = \sum_{a \in A} \underbrace{\mathbb{E}[G \mid A = a, T_t]}_{\text{value of the action}} \underbrace{\mathbb{P}(A = a \mid T_t)}_{\text{likelihood of choosing it}}$$

Each factor is then expanded again — passes over destination and over success/failure — until nine estimable pieces remain, each fit independently and multiplied back together.

The decomposition is exact. No approximation is introduced by the algebra; the approximation lives entirely in the component models.

## Why Bother

### Interpretability without simplification

This is the main argument, and it cuts against a pattern the vault sees everywhere else. In [[action-valuation-frameworks-compared|the valuation comparison]], richer state consistently buys sensitivity at the cost of interpretability — [[expected-threat|xT]] is legible because it is simple, [[vaep]] and [[martingale-epv]] are opaque because they are not.

Decomposition offers a third option: keep the rich model, but split it along lines a domain expert already thinks in. A coach shown "passing is worth 0.0252 and 29.3% likely; shooting is worth 0.0242 and 47.6% likely" can interrogate the estimate. Shown only "EPV = 0.0239" they cannot.

The lines of the split matter more than the fact of splitting. The decomposition works here because *pass / ball drive / shoot* is how the sport is already discussed. A mathematically valid decomposition along axes nobody thinks in would buy nothing.

### Component-specific modelling

Each subproblem gets the architecture that suits it. Pass components need full-field [[probability-surface|surfaces]] and use [[soccermap|fully convolutional networks]]; ball-drive and shot components need single-point predictions and use shallow dense networks. A monolithic model would have to compromise.

### Reusability

Each component is independently useful. The pass success surface is a deliverable in its own right; so is pass selection. The paper's applications draw on individual components far more than on the joint estimate.

## The Risk: Compounding Error

The obvious objection is that errors multiply. Nine models, each imperfect, recombined — why should the product be trustworthy?

The paper's answer is empirical and is its central validation claim: the joint estimate's [[probability-calibration|expected calibration error]] (0.0023) is comparable to that of its components (0.0011–0.0095), and the joint loss (0.0078) sits within the range of the component EPV losses. Decomposition did not degrade the whole.

This should not be over-generalised. It holds because the components are individually well-calibrated, and calibration is exactly the property preserved under this kind of probability-weighted combination. A decomposition of poorly-calibrated components would compound rather than cancel — **calibration of the parts is what licenses trust in the whole**, and it needs checking rather than assuming.

## Contrast with Related Ideas

| | Structured decomposition | Ensembling | Multi-task learning |
|---|---|---|---|
| Components estimate | *Different* quantities | The *same* quantity | Related quantities |
| Combination | Algebraic identity | Averaging / voting | Shared representation |
| Motivation | Interpretability, specialisation | Variance reduction | Data efficiency |

The distinction from ensembling is worth keeping sharp: an ensemble's members are interchangeable and individually meaningless, while a decomposition's components each answer their own question. See [[multi-task-learning]] for the third column.

## Elsewhere

The pattern recurs wherever a quantity factorises along an interpretable axis: hazard decomposition into cause-specific components ([[competing-risks]]), the chain-rule factorisation underlying every [[autoregressive-model]], and [[multiresolution-modelling|macro/micro transition splits]] in the basketball EPV model — which is a decomposition by *time scale* rather than by action type.

## See Also

- [[expected-possession-value]] · [[soccermap]] · [[probability-surface]]
- [[probability-calibration]] · [[policy-modelling]] · [[interpretability]]
- [[multiresolution-modelling]] · [[competing-risks]] · [[multi-task-learning]]
- [[expected-value-possession-framework|Source Summary]]
