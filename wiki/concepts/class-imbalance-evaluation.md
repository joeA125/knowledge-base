---
title: "Class-Imbalance Evaluation"
type: concept
tags: [class-imbalance, evaluation, machine-learning, statistics, calibration, probabilistic-classification, proxy-target, sports-analytics]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 30%
  generated: 10%
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

A classifier predicting *nothing ever happens* achieves accuracy $\approx 0.992$, an excellent **Brier score** (squared error against a near-zero base rate is tiny), and a respectable **AUC** (ranking is barely tested by so few positives).

[[vaep]]'s conceding classifier records **Brier = 0.003** — the best in the comparison — alongside **F1 = 0.000**.

## What Each Metric Actually Rewards

| Metric | Uses true negatives? | Under heavy imbalance |
|---|---|---|
| Accuracy | Yes, dominantly | Approaches the base rate; uninformative |
| **Brier** | Yes | **Rewards predicting the rarer event** |
| **AUC** | Yes (via false-positive rate) | Inflated; ranking barely constrained |
| **F1** | **No** | Collapses to 0 if no true positives are found |
| Precision–recall AUC | No | Preferred alternative to ROC-AUC here |

The Brier row explains why the VDEP comparison inverts across metrics: **VAEP wins on Brier, VDEP wins on AUC and overwhelmingly on F1.** VAEP looks better on Brier precisely *because* its target is rarer.

F1's virtue is what it ignores. Built only from true positives, false positives and false negatives, it never sees the true-negative mass that inflates everything else.

## ⚠️ The Converse: F1 Can Manufacture a Failure

The argument above is one-directional and needs its complement.^[generated: this objection is raised here; no held source makes it. rests-on: source:vdep-f1-table, source:vaep-never-thresholds]

**F1 requires hard predictions**, conventionally thresholded at 0.5. A *well-calibrated* classifier on a 0.23% base rate emits probabilities overwhelmingly below 0.05, crosses 0.5 almost never, predicts no positives, and scores **F1 = 0.000 by construction** — however well it discriminates.

VAEP's conceding classifier shows AUC 0.701 — not chance — and the best calibration in the comparison. And **VAEP never thresholds**: it computes $\Delta P_{concedes}$, a difference of two probabilities.

Two pieces of independent support have since arrived:

**[[gvdep|GVDEP]] reports concedes F1 of 0.08–0.15** on comparable data at a marginally rarer base rate. So the zero is not a fixed property of the model — it moves with implementation, which is what a threshold artefact looks like where a degenerate model would be reliably zero.

**[[physics-based-pass-probabilities|Spearman et al. (2017)]] demonstrate the mechanism directly.** Their pass model's accuracy rises from **80.5% to 81.9% simply by moving the success cutoff from 0.5 to 0.27**, because most passes succeed. Nothing about the model changed. The threshold is a convention, and choosing it badly costs real measured performance — on a *majority*-class problem, where the effect is mild. Under football's conceding base rate the same mechanism is catastrophic rather than mild.

See [[vaep-conceding-classifier]].

## The General Rule

**Choosing an evaluation metric is choosing a use case.**^[generated: framing constructed here]

| The model's output is used as… | Evaluate with |
|---|---|
| A **decision** (act / do not act) | F1, precision–recall, the confusion matrix — **at a base-rate-appropriate threshold** |
| A **probability** (summed, differenced, weighted) | Calibration, Brier, PR-AUC, threshold-free comparisons |
| A **ranking** (top-$k$ retrieval) | AUC, precision@$k$ |

F1 is the right correction when a model is used to decide and the base rate is low. It is the wrong instrument when the output feeds arithmetic — which is what [[vaep]], [[vdep]] and [[expected-value-possession-framework|Fernández et al.]] all do.

**And where a threshold is genuinely needed, 0.5 should be justified rather than assumed.** Spearman et al. tune theirs and report the gain; nobody in the valuation literature does.

If two models must be compared and neither thresholds, **PR-AUC is the fair choice**.

## Calibration Is Not Enough Either

A model predicting the base rate for every input is **perfectly calibrated** and completely uninformative. Three checks are needed, not two:

- **Calibration** — do the numbers mean what they say?
- **Discrimination** — does the model separate positives from negatives at all? (AUC, PR-AUC)
- **Variation** — does the output *move* across inputs, or has it collapsed to a constant?

The third is the one nobody reports and the one that would settle the VAEP question directly: the standard deviation of $\Delta P_{concedes}$ relative to $\Delta P_{scores}$.

## Practical Guidance

- **Report the base rate.** An accuracy of 0.99 means nothing without it.
- **Show the confusion matrix**, where F1 = 0.000 becomes legible as "never predicts positive".
- **Report a true-negative-insensitive metric** — but check first whether the model is ever thresholded.
- **Tune the threshold, or justify 0.5.** It is a free parameter treated as a constant.
- **Do not compare models with different target frequencies on Brier.**
- **Consider changing the target.** See [[rare-event-proxy-targets]].

## Relation to Training

Under imbalance an unweighted loss is dominated by the majority class and gradient descent finds the degenerate optimum. Remedies — class weighting, resampling, focal loss — parallel [[sample-weighting]], which addresses uneven representation across *groups* rather than across *labels*.

Both are the same structural point: **an unweighted objective optimises for whatever the data contains most of.**

## See Also

- [[vaep-conceding-classifier]] — the open question this page's converse raises
- [[probability-calibration]] · [[rare-event-proxy-targets]] · [[sample-weighting]] · [[probabilistic-classification]]
- [[vdep]] · [[gvdep]] · [[vaep]] · [[defensive-valuation]] · [[expected-goals]] · [[predictive-validity]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[evaluating-football-player-actions|VAEP Summary]] · [[physics-based-pass-probabilities|Spearman 2017 Summary]]
