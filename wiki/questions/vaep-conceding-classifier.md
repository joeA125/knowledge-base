---
title: "Is VAEP's conceding classifier broken, or just unthresholdable?"
type: question
tags: [vaep, class-imbalance, probabilistic-classification, defensive-valuation, evaluation, probability-calibration, model-selection, needs-review]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluating-football-player-actions.md, raw/papers/defensive_player_location_analysis.md, raw/papers/physics_based_pass_probabilities.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 45%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Is VAEP's conceding classifier broken, or just unthresholdable?

**Status:** Open, but **substantially narrowed**. Two independent pieces of evidence now support the thresholding explanation, and the strongest reading is ruled out.

## The Finding

[[football-defence-evaluation-vdep|Toda et al. (2022)]] re-implement [[vaep]] on 45 J-League matches:

| Classifier | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | 0.003 | **0.000** |

227 conceding events in 97,335. The vault recorded this as VAEP's defensive half being *empirically inert*, across four pages.

## The Objection

**F1 requires hard predictions**, conventionally thresholded at 0.5.

A *well-calibrated* classifier on a 0.23% base rate emits probabilities overwhelmingly below 0.05. Almost nothing crosses 0.5. It predicts no positives and scores **F1 = 0.000 by construction**, however well it discriminates.

This model does discriminate — **AUC 0.701**, not chance — and has the best calibration in the comparison (Brier 0.003).

**VAEP never thresholds.** It computes $\Delta P_{concedes}$, a difference of two probabilities. Hard classification appears nowhere in the pipeline, so the reported failure may be of a use case that does not exist.

## Two Pieces of Independent Support

### 1. The zero is not universal

[[gvdep|Umemoto, Tsutsui & Fujii (2022)]] train the same VAEP classifiers on Euro 2020 — 186 conceding events in 100,328, a base rate of 0.19%, marginally *rarer* than Toda's. They report concedes F1 between **0.08 and 0.15**.

**The strongest reading is ruled out.** "VAEP's conceding classifier is inert" cannot be a fixed property of the model, since the same model on comparable data produces non-zero F1. Toda's 0.000 was one dataset's outcome.

**The thresholding explanation gains support.** F1 hovering near zero and moving with implementation details is what an artefact looks like. A genuinely degenerate model would be reliably zero.

### 2. The mechanism, demonstrated directly

> **Added 2026-07-27** on ingest of [[physics-based-pass-probabilities|Spearman et al. (2017)]].

Their pass-completion model reports accuracy of **80.5% at the conventional 0.5 cutoff, rising to 81.9% when the cutoff is moved to 0.27** — because most passes succeed, so 0.5 sits in the wrong place relative to the base rate. Nothing about the model changed; only the threshold did.

That is the mechanism operating **in the open**, on a majority-class problem where its effect is mild and merely costs 1.4 points of accuracy. Football's conceding base rate is 0.23%, roughly 340× more extreme in the opposite direction. The same mechanism there is not mild — it is the difference between a model that appears to find nothing and one that is never asked to.

This matters because it converts the objection from an analytical argument into an **observed effect in the same literature**, on a model whose authors noticed and corrected for it. Nobody in the valuation line does.

## What Still Indicts the Model

**VAEP correlates $\approx 0$ with goals conceded** — $r = -0.098$ across a season, $-0.040$ within a match — despite being built from a conceding classifier. That is a failure of the *quantity VAEP actually uses*, and no thresholding artefact explains it.

A second, from GVDEP: **concedes F1 gets worse as more player information is added** (0.15 → 0.08). With 186 positives, extra dimensions are noise. That is small-data overfitting, not a threshold effect, and it is independent evidence the classifier genuinely struggles.

So the conclusion may be right while the headline diagnostic is wrong — a worse position than either being simply right or simply wrong.

## The Test That Actually Bears On It

The question is not "what is F1 at scale" but **does $\Delta P_{concedes}$ vary meaningfully across actions?**

1. **Report the distribution of $\Delta P_{concedes}$** — its standard deviation relative to $\Delta P_{scores}$. One number. If tiny, the defensive half is decorative regardless of any classification metric.
2. **Ablate it.** Recompute VAEP as $\Delta P_{scores}$ alone and compare player rankings. If unchanged, the defensive term is inert *in the sense that matters*.
3. **F1 at a base-rate-appropriate threshold**, on the original 8.5M-action corpus, alongside a precision–recall curve. Spearman et al. supply the protocol: tune the cutoff and report the gain.
4. **Threshold-free comparison.** Re-run Toda et al.'s comparison using PR-AUC. If VDEP still beats VAEP, the finding survives the reframe and is strengthened by it.

Steps 1 and 2 are cheap and decisive about VAEP. Step 3 is now well-specified rather than vague — the 2017 paper shows exactly what tuning a threshold looks like and what it is worth reporting.

## What Would Change

**If $\Delta P_{concedes}$ is near-constant** — the vault's claim stands, stated correctly: VAEP's defensive term contributes no variation. Stronger and better-founded than "F1 = 0.000".

**If it varies but rankings do not change** — the term is real but immaterial, and VAEP is effectively an offensive metric wearing a risk-aware label. That would qualify its classification as *action-based*, since risk modelling is the stated basis for it.

**If it varies and matters** — the near-zero correlation with conceded goals needs another explanation, and the VAEP-versus-VDEP comparison needs redoing on threshold-free terms.

## The General Lesson

A **metric applied to a model that is not used that way**. F1 presumes a decision; VAEP makes none. [[class-imbalance-evaluation]] argues correctly that F1 exposes failures the Brier score hides — and the converse also applies: **F1 can manufacture a failure in a model that was never going to be thresholded.**

Choosing an evaluation metric is choosing a use case. And where a threshold is genuinely needed, **0.5 is a convention, not a property of the model** — a free parameter almost universally treated as a constant.

## See Also

- [[vaep]] · [[vdep]] · [[gvdep]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[probabilistic-classification]]
- [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[action-valuation]] · [[model-selection]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[physics-based-pass-probabilities|Spearman 2017 Summary]]
