---
title: "Probability Calibration"
type: concept
tags: [machine-learning, calibration, probabilistic-classification, evaluation, statistics, model-decomposition, uncertainty-quantification, deep-learning, class-imbalance]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/expected_value_possession_framework.md, raw/papers/llm_factcheck_consistency_certainty.md, raw/papers/football_defence_evaluation.md]
confidence: 0.9
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
---

# Probability Calibration

A probabilistic classifier is well-calibrated if its predicted probabilities match observed frequencies: among all events assigned probability 0.3, roughly 30% should actually occur. Calibration is distinct from **discrimination** (ranking positives above negatives) — a model can rank perfectly yet be poorly calibrated, or vice versa.

## When It Becomes the Binding Constraint

Calibration matters more than discrimination whenever probabilities are used as **quantities** rather than as an ordering. Two cases here, both from football valuation:

**Arithmetic on probabilities.** [[vaep]] computes action values by summing and subtracting predictions:

$$V(a_i) = \Delta P_{scores}(a_i) - \Delta P_{concedes}(a_i)$$

Miscalibration propagates straight into the action values. Ranking quality alone (high AUC) is insufficient.

**Recombining a decomposition.** [[expected-value-possession-framework|Fernández, Bornn & Cervone]] estimate nine components separately and multiply them back together. Here calibration is load-bearing — see below.

## Evaluation Metrics

### Brier Score
Mean squared error between predicted probabilities and binary outcomes:

$$BS = \frac{1}{N}\sum_{i=1}^{N}(p_i - o_i)^2$$

A **proper scoring rule** — minimised when the true distribution is reported, so sensitive to both calibration and discrimination. VAEP's CatBoost model achieves 0.01376 for scoring probability.

### Expected Calibration Error (ECE)
Bin the predictions, then average the gap between mean prediction and mean outcome per bin, weighted by bin size:

$$\text{ECE} = \sum_{k=1}^{K} \frac{|B_k|}{N} \left| \frac{1}{|B_k|}\sum_{i \in B_k} y_i - \frac{1}{|B_k|}\sum_{i \in B_k} \hat{y}_i \right|$$

ECE isolates calibration where Brier conflates it with discrimination — a model can improve its Brier score by ranking better while getting no closer to honest probabilities. Quantile binning is preferable to equal-width, which leaves sparse tail bins dominated by noise.

Used by [[expected-value-possession-framework|Fernández et al.]] on every component, and by [[llm-factcheck-consistency-certainty|the PCC fact-checking paper]] in an LLM setting.

### Reliability curves
Mean prediction against mean outcome per bin, with point size showing bin mass. More informative than ECE alone: it shows *where* along the range a model is miscalibrated, and reveals when a model never emits predictions in some range at all.

## ⚠️ Calibration Is Not Sufficient

The most important caveat, and one that only becomes visible under class imbalance.

**A model that always predicts the base rate is perfectly calibrated and completely uninformative.** Calibration checks that stated probabilities match frequencies; it says nothing about whether the model distinguishes cases.

[[football-defence-evaluation-vdep|Toda et al.]] supply the concrete instance. VAEP's conceding classifier records the **best Brier score in their comparison (0.003)** alongside an **F1 of exactly 0.000** — it identifies no true positives whatsoever, having learned to predict "no goal" always, correct 99.2% of the time. Every true-negative-sensitive metric rewards it; only F1, which ignores true negatives, exposes the failure.

Worse, the same effect makes cross-model comparison actively misleading: a classifier looks better on Brier simply for having a **rarer target**. Comparing VAEP against [[vdep]] on Brier inverts the correct conclusion.

So calibration belongs alongside discrimination, not in place of it:

| Check | Question |
|---|---|
| **Calibration** (ECE, Brier, reliability curve) | Do the numbers mean what they say? |
| **Discrimination under imbalance** (F1, PR-AUC, confusion matrix) | Does the model find the rare positives at all? |

See [[class-imbalance-evaluation]] for the full treatment.

## Achieving Calibration

Some model classes are naturally better calibrated. [[gradient-boosting|Gradient-boosted trees]] and well-regularised models tend to produce reasonable probabilities; plain SVMs and many neural networks do not.

**Modern neural networks are systematically overconfident** (Guo et al., 2017) — counter to the intuition that better accuracy brings better probabilities.

### Temperature scaling
Divide the logits by a learned scalar $T$ before the softmax or sigmoid. Because it is monotonic, it **changes the probability distribution while leaving every ranking untouched** — accuracy, AUC and the argmax prediction are provably unaffected. A single parameter fit on a validation set that can only help calibration.

Fernández et al. obtain $T = 0.82$ for pass success and $T = 0.5$ for pass selection. Both below 1, which *sharpens* rather than softens — the opposite of the usual overconfidence correction, suggesting those two models were under-confident.

Alternatives: Platt scaling (a logistic fit, more flexible, more prone to overfitting) and isotonic regression (nonparametric, needs more validation data).

## Calibration Under Composition

The most interesting result the vault holds on this topic, and the reason [[structured-model-decomposition]] is defensible at all.

If nine separately-fit models are multiplied together, does the product mean anything? Fernández et al. answer empirically: the joint EPV estimate's ECE is **0.0023**, comfortably inside the range of its components (0.0011–0.0095), and the joint loss (0.0078) sits within the component losses.

The mechanism matters more than the number. Combination here is a **probability-weighted sum of conditional expectations** — the law of total expectation. If each conditional expectation is unbiased and each weight is a calibrated probability, the combination inherits unbiasedness. Errors average rather than accumulate.

The caveat is that this depends entirely on the parts being calibrated. **A decomposition of overconfident components compounds rather than cancels**, since systematic bias in the same direction multiplies through. Calibration of the parts licenses trust in the whole, and needs checking rather than assuming — which is why reporting per-component ECE is the right practice and an uncommon one.

## A Limit Worth Knowing

Calibration can only be verified where outcomes are observed. For a [[probability-surface]] this is a real restriction: only one cell per training example has ground truth, so reported ECE speaks to the thin diagonal of *observed* pass destinations and says nothing about the rest of the surface. See [[single-pixel-supervision]].

## See Also

- [[class-imbalance-evaluation]] · [[rare-event-proxy-targets]]
- [[vaep]] · [[vdep]] · [[expected-possession-value]] · [[structured-model-decomposition]]
- [[gradient-boosting]] · [[uncertainty-quantification]]
- [[probability-surface]] · [[single-pixel-supervision]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[expected-value-possession-framework|Soccer EPV Framework Summary]]
- [[llm-factcheck-consistency-certainty|PCC Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
