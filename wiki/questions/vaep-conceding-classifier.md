---
title: "Is VAEP's conceding classifier broken, or just unthresholdable?"
type: question
tags: [vaep, class-imbalance, probabilistic-classification, defensive-valuation, evaluation, probability-calibration, model-selection, needs-review]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.7
provenance:
  extracted: 40%
  inferred: 55%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Is VAEP's conceding classifier broken, or just unthresholdable?

**Status:** Open. The vault has treated this as settled and it is not — the headline diagnostic may be measuring something [[vaep|VAEP]] never does.

## The Finding

[[football-defence-evaluation-vdep|Toda et al. (2022)]] re-implement VAEP on 45 J-League matches and report:

| Classifier | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | 0.003 | **0.000** |

227 conceding events in 97,335. The vault has recorded this as the defensive half of VAEP being *empirically inert*, and it appears on [[vaep]], [[action-valuation]], [[defensive-valuation]] and the [[action-valuation-frameworks-compared|synthesis]].

## The Objection I Should Have Raised Earlier

**F1 requires hard predictions.** It is computed by thresholding — conventionally at 0.5 — and counting true and false positives.

A *well-calibrated* classifier on a 0.23% base rate will emit probabilities overwhelmingly below 0.05. Almost nothing will cross 0.5. It will therefore predict no positives and score **F1 = 0.000 by construction**, however well it discriminates.

And this model does discriminate somewhat: **AUC 0.701**, which is not chance. It is also well calibrated — Brier 0.003, the best in the comparison.

So F1 = 0.000 is consistent with two very different states:

1. The classifier has learned nothing beyond the base rate.
2. The classifier has learned something, is properly calibrated, and F1 is the wrong instrument for a model used as a **probability**.

**VAEP never thresholds.** It computes $\Delta P_{concedes}$ — a difference of two probabilities. Hard classification is not part of the pipeline at any point. So the reported failure may be of a use case that does not exist.

## What Still Indicts It

The reframe above does not exonerate the model, because there is independent evidence.

**VAEP correlates ≈ 0 with goals conceded** — $r = -0.098$ across a season, $-0.040$ within a match — despite being constructed from a conceding classifier. That is a failure of the *quantity VAEP actually uses*, and no thresholding artefact explains it.

So the conclusion may be right while the headline diagnostic is wrong. That is a worse position than either being simply right or simply wrong, because it means the vault's most-cited critique rests on a metric that does not test the claim.

## The Test That Actually Bears On It

The question is not "what is F1 at scale" but **does $\Delta P_{concedes}$ vary meaningfully across actions?**

If the classifier has collapsed to the base rate, $P_{concedes}$ is near-constant, so $\Delta P_{concedes} \approx 0$ for every action and VAEP reduces in practice to its offensive term. If it varies, the defensive term is contributing something and the near-zero correlation with conceded goals needs a different explanation.

1. **Report the distribution of $\Delta P_{concedes}$** — its standard deviation relative to $\Delta P_{scores}$. One number. If the ratio is tiny, the defensive half is decorative regardless of any classification metric.
2. **Ablate it.** Recompute VAEP as $\Delta P_{scores}$ alone and compare player rankings. If rankings are unchanged, the defensive term is inert *in the sense that matters*.
3. **Then, and only then, F1 at scale.** [[evaluating-football-player-actions|Decroos et al.]] trained on 8.5M actions — roughly 75× more conceding events — and never reported F1. It would show whether discrimination improves with positives, but should be read alongside a **precision–recall curve** and a base-rate-appropriate threshold, not at 0.5.
4. **Threshold-free comparison.** Re-run Toda et al.'s comparison using PR-AUC rather than F1. If VDEP still beats VAEP there, the finding survives the reframe intact and is strengthened by it.

Steps 1 and 2 are cheap and decisive about VAEP. Step 4 is the fair version of the original comparison.

## What Would Change

**If $\Delta P_{concedes}$ is near-constant** — the vault's claim stands, stated correctly: VAEP's defensive term contributes no variation, which is a stronger and better-founded statement than "F1 = 0.000."

**If it varies but rankings do not change** — the term is real but immaterial, and VAEP is effectively an offensive metric wearing a risk-aware label. That would qualify its classification as *action-based* rather than possession-based, since risk modelling is the stated basis for that classification.

**If it varies and matters** — then the near-zero correlation with conceded goals needs another explanation, and the whole VAEP-versus-VDEP comparison needs redoing on threshold-free terms.

## The General Lesson, Regardless of Outcome

This is a case of a **metric being applied to a model that is not used that way**. F1 presumes a decision; VAEP makes none. The vault's [[class-imbalance-evaluation]] page argues correctly that F1 exposes failures the Brier score hides — and that argument holds — but the converse also applies: **F1 can manufacture a failure in a model that was never going to be thresholded.**

Choosing an evaluation metric is choosing a use case. Both pages should say so.

## See Also

- [[vaep]] · [[vdep]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[probabilistic-classification]]
- [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[action-valuation]] · [[model-selection]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[evaluating-football-player-actions|VAEP Summary]]
- [[action-valuation-frameworks-compared]]
