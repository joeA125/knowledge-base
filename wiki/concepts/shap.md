---
title: "SHAP (SHapley Additive exPlanations)"
type: concept
tags: [feature-attribution, interpretability, machine-learning, statistics, gradient-boosting, evaluation]
sources: [raw/papers/football_defence_evaluation.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# SHAP (SHapley Additive exPlanations)

A method (Lundberg & Lee, 2017) for attributing a model's prediction to its input features, by treating the features as players in a cooperative game and computing each one's **Shapley value** — its average marginal contribution across all possible orderings of the others.

## The Idea

For a single prediction, SHAP produces an additive decomposition:

$$f(x) = \phi_0 + \sum_{j=1}^{M} \phi_j$$

where $\phi_0$ is the base value (the model's average output) and $\phi_j$ is feature $j$'s contribution to *this* prediction. The contributions sum exactly to the prediction — the "additive" in the name.

The Shapley value comes from cooperative game theory and is the unique attribution satisfying a small set of desirable axioms (efficiency, symmetry, dummy, additivity). That uniqueness is SHAP's main theoretical selling point: given those axioms, there is no alternative allocation to argue about.

Its main practical cost is that exact computation is exponential in the number of features. Tree-specific algorithms (TreeSHAP) make it tractable for [[gradient-boosting|boosted ensembles]] and [[random-forest|forests]], which is why SHAP appears most often alongside those models.

## Local, Then Global

SHAP is fundamentally a **local** method — it explains one prediction. Global summaries are built by aggregating local values across a dataset, which is what produces the familiar beeswarm plot: one row per feature, one dot per instance, position showing contribution and colour showing feature value.

The aggregation is where care is needed. A feature can have large SHAP values in both directions and average to nearly zero, so ranking by *mean* attribution hides exactly the interactions worth knowing about. Ranking by mean **absolute** value is the usual fix and is what most published plots show.

## Use in the Vault

[[football-defence-evaluation-vdep|Toda et al.]] use SHAP to check whether [[vdep]]'s off-ball features are doing real work or merely padding a 139-dimensional vector. The answer is that they dominate:

| Target | Top contributors |
|---|---|
| $P_{recoveries}$ | Distance from ball of the **nearest defender**; whether possession changed on the previous event |
| $P_{attacked}$ | **$x$-position of the nearest attacker** (how advanced); that attacker's displacement over the event |

Both leading features are positional and off-ball — invisible to on-ball frameworks like [[vaep]]. This is the paper's strongest evidence that the extra state representation, rather than the change of target, is carrying part of the improvement.

It also yields a sanity check of the kind worth borrowing: the recovered attributions are *football-plausible*. A defence recovers the ball when someone is close to it; a defence is penetrated when an attacker gets high up the pitch. A model whose attributions contradicted domain intuition would warrant suspicion regardless of its metrics.

## Caveats

- **Attribution is not causation.** SHAP explains what the *model* used, not what *drives the world*. A model leaning on a confounded feature gets a confident, correct-looking SHAP plot.
- **Correlated features share credit unstably.** With 22 player positions and 22 distances, many features are near-duplicates; Shapley values split credit among them in ways that shift with small changes to the model.
- **The background distribution matters.** "Marginal contribution" is relative to some reference; different choices give different attributions for the same prediction.
- **Plausibility is not validation.** That attributions look sensible is weak evidence — it confirms the model has not learned something absurd, not that it has learned something true.

## Relation to Other Interpretability Routes

SHAP is *post-hoc* — applied to a fitted black box. The vault's other route is **structural**: [[structured-model-decomposition|decomposing a model]] so its parts are individually meaningful, as [[expected-value-possession-framework|Fernández et al.]] do.

The two answer different questions. SHAP says *which inputs moved this prediction*; decomposition says *which sub-quantities compose it*. A coach asking "why is this possession valuable?" is better served by the second; a modeller asking "is my feature set justified?" by the first.

See [[interpretability]] for the broader distinction.

## See Also

- [[interpretability]] · [[structured-model-decomposition]]
- [[gradient-boosting]] · [[random-forest]] · [[feature-engineering]]
- [[vdep]] · [[defensive-valuation]] · [[off-ball-value]]
- [[football-defence-evaluation-vdep|Source Summary]]
