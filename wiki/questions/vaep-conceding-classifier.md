---
title: "Is VAEP's conceding classifier broken, or just unthresholdable?"
type: question
tags: [vaep, class-imbalance, probabilistic-classification, defensive-valuation, evaluation, probability-calibration, model-selection, needs-review]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 50%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Is VAEP's conceding classifier broken, or just unthresholdable?

**Status:** Open, but **narrowed** — a second independent measurement now exists, and it rules out the strongest reading.

## The Finding

[[football-defence-evaluation-vdep|Toda et al. (2022)]] re-implement [[vaep]] on 45 J-League matches:

| Classifier | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | 0.003 | **0.000** |

227 conceding events in 97,335. The vault recorded this as the defensive half of VAEP being *empirically inert*, across four pages.

## The Objection

**F1 requires hard predictions.** It is computed by thresholding — conventionally at 0.5 — and counting true and false positives.

A *well-calibrated* classifier on a 0.23% base rate will emit probabilities overwhelmingly below 0.05. Almost nothing crosses 0.5. It will therefore predict no positives and score **F1 = 0.000 by construction**, however well it discriminates.

This model does discriminate somewhat — **AUC 0.701**, not chance — and is well calibrated, with the best Brier score in the comparison.

**VAEP never thresholds.** It computes $\Delta P_{concedes}$, a difference of two probabilities. Hard classification appears nowhere in the pipeline, so the reported failure may be of a use case that does not exist.

## New Evidence: The Zero Is Not Universal

> **Updated, 2026-07-27** on ingest of [[generalized-vdep-euro-location-analysis|GVDEP]].

[[gvdep|Umemoto, Tsutsui & Fujii (2022)]] train the same VAEP classifiers on Euro 2020 data — 186 conceding events in 100,328, a base rate of 0.19%, marginally *rarer* than Toda's. They report concedes F1 between **0.08 and 0.15** depending on how many players are included.

Non-zero, but very low. Three things follow:

**The strongest reading is ruled out.** "VAEP's conceding classifier is inert" cannot be a fixed property of the model, since the same model on comparable data produces non-zero F1. Toda's 0.000 was one dataset's outcome, not a constant.

**The thresholding explanation gains support.** F1 hovering near zero and moving with implementation details is what a threshold artefact looks like. A genuinely degenerate model would be reliably zero.

**But the classifier is still weak.** 0.08–0.15 is poor by any standard, so the reframe rescues the *diagnostic* without rescuing the *model*.

An additional detail from the same source cuts deeper: **the concedes F1 gets worse as more player information is added** — 0.15 at zero players falling to 0.08 at eleven. With 186 positives, extra dimensions are noise. That is a small-data overfitting result, not a thresholding artefact, and it is independent evidence that the classifier is genuinely struggling.

## What Still Indicts It

**VAEP correlates $\approx 0$ with goals conceded** — $r = -0.098$ across a season, $-0.040$ within a match — despite being constructed from a conceding classifier. That is a failure of the *quantity VAEP actually uses*, and no thresholding artefact explains it.

So the conclusion may be right while the headline diagnostic is wrong — a worse position than either being simply right or simply wrong, because the vault's most-cited critique rested on a metric that does not test the claim.

## The Test That Actually Bears On It

The question is not "what is F1 at scale" but **does $\Delta P_{concedes}$ vary meaningfully across actions?**

1. **Report the distribution of $\Delta P_{concedes}$** — its standard deviation relative to $\Delta P_{scores}$. One number. If the ratio is tiny, the defensive half is decorative regardless of any classification metric.
2. **Ablate it.** Recompute VAEP as $\Delta P_{scores}$ alone and compare player rankings. If rankings are unchanged, the defensive term is inert *in the sense that matters*.
3. **F1 at scale.** [[evaluating-football-player-actions|Decroos et al.]] trained on 8.5M actions and never reported F1. Read alongside a **precision–recall curve** and a base-rate-appropriate threshold, not 0.5.
4. **Threshold-free comparison.** Re-run Toda et al.'s comparison using PR-AUC. If VDEP still beats VAEP, the finding survives the reframe and is strengthened by it.

Steps 1 and 2 are cheap and decisive about VAEP. Step 4 is the fair version of the original comparison.

## What Would Change

**If $\Delta P_{concedes}$ is near-constant** — the vault's claim stands, stated correctly: VAEP's defensive term contributes no variation, which is stronger and better-founded than "F1 = 0.000".

**If it varies but rankings do not change** — the term is real but immaterial, and VAEP is effectively an offensive metric wearing a risk-aware label. That would qualify its classification as *action-based*, since risk modelling is the stated basis for it.

**If it varies and matters** — the near-zero correlation with conceded goals needs another explanation, and the VAEP-versus-VDEP comparison needs redoing on threshold-free terms.

## The General Lesson

This is a case of a **metric being applied to a model that is not used that way**. F1 presumes a decision; VAEP makes none. [[class-imbalance-evaluation]] argues correctly that F1 exposes failures the Brier score hides — and the converse also applies: **F1 can manufacture a failure in a model that was never going to be thresholded.**

Choosing an evaluation metric is choosing a use case.

## See Also

- [[vaep]] · [[vdep]] · [[gvdep]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[probabilistic-classification]]
- [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[action-valuation]] · [[model-selection]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[evaluating-football-player-actions|VAEP Summary]]
