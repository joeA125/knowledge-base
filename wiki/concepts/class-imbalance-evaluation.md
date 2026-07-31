---
title: "Class-Imbalance Evaluation"
type: concept
tags: [class-imbalance, evaluation, machine-learning, statistics, calibration, probabilistic-classification, proxy-target, sports-analytics]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 32%
  generated: 13%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Class-Imbalance Evaluation

Choosing metrics when one label is far rarer than the other. Under severe imbalance the standard metrics stop measuring what their names suggest — and, less obviously, **the metrics recommended as corrections can manufacture failures of their own.**

## The Failure, Concretely

[[football-defence-evaluation-vdep|Toda et al.]] supply a clean demonstration. Across 97,335 events there are 753 positive "scored" labels and 227 "conceded".

A classifier predicting *nothing ever happens* achieves:

- Accuracy $1 - 753/97{,}335 \approx \mathbf{0.992}$
- An excellent **Brier score**, since squared error against a near-zero base rate is tiny
- A respectable **AUC**, since ranking is barely tested by so few positives

[[vaep]]'s conceding classifier records **Brier = 0.003** — the best score in the comparison table — alongside **F1 = 0.000**.

## What Each Metric Actually Rewards

| Metric | Uses true negatives? | Under heavy imbalance |
|---|---|---|
| Accuracy | Yes, dominantly | Approaches the base rate; uninformative |
| **Brier** | Yes | **Rewards predicting the rarer event** — smaller errors on more zeros |
| **AUC** | Yes (via false-positive rate) | Inflated; ranking barely constrained by few positives |
| **F1** | **No** | Collapses to 0 if no true positives are found |
| Precision–recall AUC | No | Preferred alternative to ROC-AUC here |

The Brier row is counter-intuitive and explains why the VDEP comparison inverts across metrics: **VAEP wins on Brier, VDEP wins on AUC and overwhelmingly on F1.** VAEP looks better on Brier precisely *because* its target is rarer. Reading that table by Brier alone reverses the correct conclusion.

$$F_1 = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$

F1's virtue is what it ignores. Built only from true positives, false positives and false negatives, it never sees the true-negative mass that inflates everything else.

## ⚠️ The Converse: F1 Can Manufacture a Failure

The argument above is one-directional and needs its complement.^[generated: this objection is raised here; no held source makes it]

**F1 requires hard predictions.** It is computed by thresholding — conventionally at 0.5 — and counting. A *well-calibrated* classifier on a 0.23% base rate emits probabilities overwhelmingly below 0.05, crosses 0.5 almost never, predicts no positives, and scores **F1 = 0.000 by construction** — however well it discriminates.

So F1 = 0.000 is consistent with two very different states:

1. The classifier has learned nothing beyond the base rate.
2. The classifier has learned something, is properly calibrated, and F1 is the wrong instrument for a model used as a **probability**.

VAEP's conceding classifier shows AUC 0.701 — not chance — and the best calibration in the comparison. And **VAEP never thresholds**: it computes $\Delta P_{concedes}$, a difference of two probabilities. Hard classification appears nowhere in the pipeline.

The finding may still be right. VAEP correlates $\approx 0$ with goals conceded ($r = -0.098$), which is a failure of the quantity VAEP actually uses and which no thresholding artefact explains. But that is *different evidence*, and the vault cited the wrong number for several entries. See [[vaep-conceding-classifier]].

## The General Rule

**Choosing an evaluation metric is choosing a use case.**^[generated: framing constructed here]

| The model's output is used as… | Evaluate with |
|---|---|
| A **decision** (act / do not act) | F1, precision–recall, the confusion matrix |
| A **probability** (summed, differenced, weighted) | Calibration, Brier, PR-AUC, and threshold-free comparisons |
| A **ranking** (top-$k$ retrieval) | AUC, precision@$k$ |

F1 is the right correction when a model is used to decide and the base rate is low. It is the wrong instrument when the model's output feeds arithmetic — which is exactly what [[vaep]], [[vdep]] and [[expected-value-possession-framework|Fernández et al.]] all do.

If two models must be compared and neither thresholds, **PR-AUC is the fair choice**: threshold-free, insensitive to true negatives, and it would let the VDEP-versus-VAEP comparison stand or fall on its merits.

## Calibration Is Not Enough Either

A model predicting the base rate for every input is **perfectly calibrated** and completely uninformative. Under imbalance, the degenerate solution is calibrated, has a superb Brier score, and finds nothing.

So three checks are needed, not two:

- **Calibration** — do the numbers mean what they say?
- **Discrimination** — does the model separate positives from negatives at all? (AUC, PR-AUC)
- **Variation** — does the output *move* across inputs, or has it collapsed to a constant?

The third is the one nobody reports and the one that would have settled the VAEP question directly: the standard deviation of $\Delta P_{concedes}$ relative to $\Delta P_{scores}$. If the defensive term is near-constant, it contributes nothing regardless of any classification metric.

## Practical Guidance

- **Report the base rate.** An accuracy of 0.99 means nothing without it.
- **Show the confusion matrix**, where F1 = 0.000 becomes legible as "the model never predicts positive".
- **Report a true-negative-insensitive metric** — but check first whether the model is ever thresholded.
- **Do not compare models with different target frequencies on Brier.** That compares base rates.
- **Consider changing the target.** If positives are too rare to learn, the metric problem is downstream of a data problem — see [[rare-event-proxy-targets]].

## Relation to Training

Evaluation and training failures share a cause. Under imbalance an unweighted loss is dominated by the majority class and gradient descent finds the degenerate optimum. Remedies — class weighting, resampling, focal loss — parallel [[sample-weighting]], which addresses uneven representation across *groups* rather than across *labels*.

Both are the same structural point: **an unweighted objective optimises for whatever the data contains most of.**

## See Also

- [[vaep-conceding-classifier]] — the open question this page's converse raises
- [[probability-calibration]] · [[rare-event-proxy-targets]] · [[sample-weighting]] · [[probabilistic-classification]]
- [[vdep]] · [[vaep]] · [[defensive-valuation]] · [[expected-goals]] · [[predictive-validity]]
- [[football-defence-evaluation-vdep|Source Summary]] · [[evaluating-football-player-actions|VAEP Summary]]
