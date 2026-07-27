---
title: "Class-Imbalance Evaluation"
type: concept
tags: [class-imbalance, evaluation, machine-learning, statistics, calibration, probabilistic-classification, proxy-target, sports-analytics]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Class-Imbalance Evaluation

Choosing metrics when one label is far rarer than the other. Under severe imbalance the standard metrics stop measuring what their names suggest, and a model can look strong by every headline number while identifying **none** of the events it was built to find.

## The Failure, Concretely

[[football-defence-evaluation-vdep|Toda et al.]] supply an unusually clean demonstration. Across 97,335 events there are 753 positive "scored" labels and 227 "conceded".

A classifier predicting *nothing ever happens* achieves:

- Accuracy $1 - 753/97{,}335 \approx \mathbf{0.992}$
- An excellent **Brier score**, since squared error against a near-zero base rate is tiny
- A respectable **AUC**, since ranking is barely tested by so few positives

And it is useless. This is not hypothetical: [[vaep]]'s conceding classifier records **Brier = 0.003** — the best score in the comparison table — alongside **F1 = 0.000**. It has learned the base rate and nothing else.

## What Each Metric Actually Rewards

| Metric | Uses true negatives? | Under heavy imbalance |
|---|---|---|
| Accuracy | Yes, dominantly | Approaches the base rate; uninformative |
| **Brier** | Yes | **Rewards predicting the rarer event** — smaller errors on more zeros |
| **AUC** | Yes (via false-positive rate) | Inflated; ranking barely constrained by few positives |
| **F1** | **No** | Collapses to 0 if no true positives are found |
| Precision–recall AUC | No | Preferred alternative to ROC-AUC here |

The Brier row is the counter-intuitive one, and the reason the VDEP comparison inverts across metrics: **VAEP wins on Brier, VDEP wins on AUC and overwhelmingly on F1.** VAEP looks better on Brier precisely *because* its target is rarer. Reading that table by Brier alone reverses the correct conclusion.

$$F_1 = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$

F1's virtue here is what it ignores. Built only from true positives, false positives and false negatives, it never sees the true-negative mass that inflates everything else.

## Calibration Is Not Enough Either

Worth stating because [[probability-calibration]] argues calibration is the binding constraint for value-differencing frameworks — which is true, and not sufficient.

A model predicting the base rate for every input is **perfectly calibrated** and completely uninformative. Calibration checks that stated probabilities match frequencies; it says nothing about whether the model distinguishes cases. Under imbalance, the degenerate solution is calibrated, has a superb Brier score, and finds nothing.

The two checks are orthogonal and both are needed:

- **Calibration** — do the numbers mean what they say?
- **Discrimination under imbalance** — does the model find the rare positives at all?

## Practical Guidance

- **Always report a metric that ignores true negatives** — F1, precision–recall AUC, or the confusion matrix itself.
- **Report the base rate.** An accuracy of 0.99 means nothing without it.
- **Show the confusion matrix.** Toda et al. include one, and it is where F1 = 0.000 becomes legible as "the model never predicts positive".
- **Do not compare across different target frequencies using true-negative-sensitive metrics.** Comparing a goal model against a recovery model on Brier is comparing base rates.
- **Consider changing the target.** If positives are too rare to learn, the metric problem is downstream of a data problem — see [[rare-event-proxy-targets]].

## Relation to Training

Evaluation and training failures share a cause. Under imbalance, an unweighted loss is dominated by the majority class and gradient descent finds the degenerate optimum. Remedies — class weighting, resampling, focal loss — parallel [[sample-weighting]], which addresses uneven representation across *groups* rather than across *labels*.

Both are the same structural point: **an unweighted objective optimises for whatever the data contains most of**, which is rarely what the modeller cares about.

## See Also

- [[probability-calibration]] · [[rare-event-proxy-targets]] · [[sample-weighting]]
- [[vdep]] · [[vaep]] · [[defensive-valuation]] · [[expected-goals]]
- [[predictive-validity]] · [[split-half-reliability]]
- [[football-defence-evaluation-vdep|Source Summary]] · [[evaluating-football-player-actions|VAEP Summary]]
