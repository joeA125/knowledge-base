---
title: "Probability Calibration"
type: concept
tags: [machine-learning, calibration, probabilistic-classification, evaluation, statistics]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 30%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Probability Calibration

A probabilistic classifier is well-calibrated if its predicted probabilities match observed frequencies: among all events assigned probability 0.3, roughly 30% should actually occur. Calibration is distinct from discrimination (ranking positives above negatives) — a model can rank perfectly yet be poorly calibrated, or vice versa.

## Why Calibration Matters for VAEP

The [[vaep]] framework computes action values by **summing and subtracting** predicted probabilities:

$$V(a_i) = \Delta P_{scores}(a_i) - \Delta P_{concedes}(a_i)$$

If the underlying scoring/conceding probabilities are miscalibrated, these arithmetic operations propagate the error directly into the action values. This is why the [[evaluating-football-player-actions|VAEP paper]] explicitly requires well-calibrated estimates — ranking quality alone (e.g. high AUC) is insufficient when the probabilities themselves are used as quantities, not just for ordering.

## Evaluation Metrics

### Brier Score
The mean squared error between predicted probabilities and binary outcomes:

$$BS = \frac{1}{N}\sum_{i=1}^{N}(p_i - o_i)^2$$

where $p_i$ is the predicted probability and $o_i \in \{0, 1\}$ the outcome. The Brier score is a **proper scoring rule** — it is minimised precisely when the true underlying probability distribution is reported, making it sensitive to both calibration and discrimination. Lower is better. VAEP's CatBoost model achieves 0.01376 for scoring probability.

### ROC AUC
The area under the receiver operating characteristic curve measures discrimination — how well the model separates positives from negatives. A key advantage the VAEP paper notes: ROC AUC is unaffected by class imbalance, important because only 1.5% of game states lead to a goal (0.5% to a concession).

## Brier Score vs AUC

The two metrics are complementary:
- **AUC** answers: "Can the model rank risky game states above safe ones?" (discrimination)
- **Brier** answers: "Are the actual probability values correct?" (calibration + discrimination)

VAEP needs both, but calibration is the binding constraint because probabilities are arithmetically combined.

## Achieving Calibration

Some model classes are naturally better calibrated than others. [[gradient-boosting|Gradient-boosted trees]] and well-regularised models tend to produce reasonable probabilities; others (e.g. plain SVMs, some neural nets) often require post-hoc calibration (Platt scaling, isotonic regression). The VAEP paper relies on CatBoost producing sufficiently calibrated outputs directly, validated via the Brier score.

## Relation to Uncertainty Quantification

Calibration is closely related to [[llm-factcheck-consistency-certainty|uncertainty quantification]] in LLMs — the PCC fact-checking paper uses Expected Calibration Error (ECE) for the same underlying goal: ensuring a model's stated confidence reflects real-world correctness frequencies.

## See Also

- [[vaep]]
- [[gradient-boosting]]
- [[event-stream-data]]
- [[evaluating-football-player-actions|Source Summary]]
