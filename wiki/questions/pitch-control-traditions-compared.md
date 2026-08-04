---
title: "Do the two pitch-control traditions agree?"
type: question
tags: [pitch-control, sports-analytics, optical-tracking-data, probability-surface, evaluation, model-selection, needs-review]
sources: [raw/papers/physics_based_pass_probabilities.md, raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.7
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

# Do the two pitch-control traditions agree?

**Status:** Open. Analytically partially resolved; empirically untested. **Reframed 2026-07-27** on ingest of the origin paper for the first tradition.

The vault holds two independent constructions of [[pitch-control]] — [[william-spearman|Spearman's]] arrival-time PPCF and [[javier-fernandez|Fernández]] & [[luke-bornn|Bornn's]] Gaussian influence model. Both are used as **inputs to value models whose outputs are compared against each other**, and no source has ever compared the surfaces themselves.

## They Are Not Answering the Same Question

> **Reframed.** This page previously treated the two as competing estimators of one quantity. [[physics-based-pass-probabilities|Spearman et al. (2017)]] shows they were built for different purposes.

**PPCF originates as a pass-reception model.** The 2017 paper asks *who will receive this pass*, fits parameters to that outcome, and derives a pitch control function by evaluating the same model for an imaginary stationary ball at every location. Control is a **by-product of reception**.

**The Gaussian influence model originates as a spatial-dominance model.** It asks *who owns this region*, with no reception event in view.

Those are related but distinct questions. A location may be one a team would reliably receive a pass at, and also one they do not meaningfully "control" in a territorial sense, and vice versa. Any disagreement between the surfaces is therefore **partly definitional rather than purely a modelling artefact** — which is a reason to expect disagreement, and a reason not to read it as one being wrong.

## The Validation Asymmetry

The sharpest difference, and it is extracted rather than inferred:

| | PPCF | Gaussian influence |
|---|---|---|
| Parameters | **Fitted by MLE**, with stat and syst errors | Set to 1, unfitted |
| Validated against | **Who actually received 5,471 held-out passes** — 81% team, 68% player | Nothing directly |
| Correctness established by | Direct prediction of an observable outcome | Downstream EPV performance only |

One tradition is an empirically validated model of a directly observable quantity. The other has never been checked against anything.

That does not make the Gaussian model wrong — it was never claimed to predict pass receivers, and it may serve its own purpose well. But it means **the two are not equally warranted**, and a disagreement between them is not symmetrically informative. If they diverge, the prior should favour PPCF wherever the question resembles reception.

## What Can Be Settled Analytically

Four structural differences; three yield directional predictions.

### 1. Saturation under crowding — the largest expected effect

$$\text{F\&B:}\quad PC = \sigma\Big(\textstyle\sum_{att} I_i - \sum_{def} I_j\Big) \qquad \text{Spearman:}\quad \frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big) f_j \lambda_j$$

Both saturate, by different mechanisms. F&B saturates through the **sigmoid**, on a *difference* of summed influences — adding a second defender to an already-dominated zone still moves the value, $\sigma(2) = 0.88$ against $\sigma(1) = 0.73$. Spearman saturates through the **shared bracket**, on *total* control: once $\sum_k PPCF_k \to 1$, every remaining contribution is multiplied by approximately zero.

**Prediction:** F&B assigns more extreme values in crowded areas. Disagreement should scale with **local player density**, worst in the penalty area and around the ball.

### 2. Attack/defence asymmetry — the cleanest to measure

Spearman (2018) fits $\kappa = 1.72$ for defenders; F&B sets $\lambda_1 = \lambda_2 = 1$. Note the 2017 paper has **no such term**, so this is a refinement of the tradition rather than a defining feature of it.

**Prediction:** a systematic shift toward defensive control in the 2018 formulation, roughly uniform across the pitch — a **bias**, visible as a non-zero mean difference before any correlation. Removable by refitting.

### 3. Offside — sharp and localised

Spearman (2018) sets $\lambda_i = 0$ for attackers in offside positions; F&B has no offside term.

**Prediction:** disagreement concentrated in a **band beyond the last defender**, near-zero elsewhere. The one place the two are not approximating the same quantity at all.

### 4. Ball travel time — direction unclear

PPCF holds control at zero until the ball could physically arrive, with flight simulated under drag. F&B treats arrival as instantaneous. Both push the same way qualitatively but by different mechanisms; the sign depends on parameter values.

## The Composite Prediction

The surfaces should agree **least where valuation depends on them most**: crowded areas near goal, and the final third around the offside line. A global correlation would therefore *understate* practical disagreement, because most of the pitch is empty and both models will agree that empty space near one team belongs to that team.

**A single global $r$ is the wrong summary.** Stratify.

## Proposed Test

Compute both surfaces on the same tracking frames — a few hundred suffice.

1. **Global agreement.** Pearson $r$ over all cells. Expect high, and expect it to mislead.
2. **Stratify by local player density.** Prediction 1 says disagreement rises monotonically.
3. **Mean signed difference**, then refit with $\kappa = 1$ and re-measure. If the shift vanishes, the asymmetry is a parameter choice rather than a structural difference.
4. **Mask the offside band** and re-run.
5. **Stratify by distance from the ball**, to characterise the travel-time effect empirically.
6. **The reception check.** Evaluate the Gaussian surface against **actual pass receivers** on held-out data, as the 2017 paper does for PPCF. This is now the most informative single test available: it would establish whether the Gaussian model predicts reception at all, and put both traditions on one directly observable criterion for the first time.
7. **The decisive step.** Recompute [[obso|OBSO]] with the Gaussian surface substituted for PPCF and compare player rankings. Surface disagreement only matters if it changes conclusions.

Step 6 is new to this revision and may be cheaper than step 7 — it needs pass outcomes rather than a full OBSO reimplementation, and the 2017 paper supplies the protocol.

## What Would Change Depending on the Answer

**If they agree closely and rankings are stable** — the traditions are notational variants, and the cheaper Gaussian model becomes the default at no cost.

**If they disagree structurally but rankings are stable** — downstream metrics are robust to their substrate, which is reassuring and non-obvious.

**If the Gaussian model predicts reception poorly** — then using it as a control term in a value model is doing something different from what PPCF does there, and the two are not substitutable even where they correlate.

**If rankings change** — differences between [[obso|OBSO]], [[c-obso]], [[drso]] and the EPV surfaces are partly artefacts of control modelling, and the vault's comparative claims about them need qualifying. Not implausible, since predicted disagreement concentrates exactly where those metrics take their largest values.

## Why Nobody Has Done It

The two traditions have **no overlapping authors and cite each other only in passing.** Fernández, Bornn & Cervone cite Spearman as related work on off-ball valuation; the [[keisuke-fujii|Fujii group]] adopts PPCF wholesale without comparing alternatives. Nobody has an incentive to test whether their own substrate is the right one.

That is the [[action-valuation-frameworks-compared|benchmarking gap]] at **component** level — sharper than the framework-level version, because a shared component that differs silently is harder to notice than two frameworks that differ openly.

## See Also

- [[pitch-control]] · [[obso]] · [[c-obso]] · [[drso]] · [[xsot]] · [[probability-surface]]
- [[expected-possession-value]] · [[off-ball-value]] · [[model-selection]] · [[shot-value-formulations-compared]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]]
- [[physics-based-pass-probabilities|Spearman 2017 Summary]] · [[beyond-expected-goals|Spearman 2018 Summary]] · [[expected-value-possession-framework|Fernández et al. Summary]]
- [[action-valuation-frameworks-compared]]
