---
title: "Do the two pitch-control traditions agree?"
type: question
tags: [pitch-control, sports-analytics, optical-tracking-data, probability-surface, evaluation, model-selection, needs-review]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.65
provenance:
  extracted: 40%
  inferred: 55%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Do the two pitch-control traditions agree?

**Status:** Open. Analytically partially resolved below; empirically untested.

The vault holds two independent constructions of [[pitch-control]] — [[william-spearman|Spearman's]] arrival-time PPCF and [[javier-fernandez|Fernández]] & [[luke-bornn|Bornn's]] Gaussian influence model. Both are used as **inputs to value models whose outputs are compared against each other**, and no source has ever compared the surfaces themselves.

This matters because a systematic difference propagates silently into [[obso|OBSO]], [[c-obso]], [[xsot|xOSOT]] and the EPV surfaces alike. If the two disagree in a structured way, then apparent differences between downstream metrics may be differences in their control substrate rather than in the valuation logic on top.

## Why This Is Answerable Cheaply

Unlike most open questions in this literature, this one needs no ground truth, no labels, and no new modelling. Both surfaces are computable from the same tracking frame. The test is a correlation.

## What Can Be Settled Analytically

The two differ on four counts. Three yield directional predictions.

### 1. Saturation under crowding — the largest expected effect

$$\text{F\&B:}\quad PC = \sigma\Big(\textstyle\sum_{att} I_i - \sum_{def} I_j\Big) \qquad\quad \text{Spearman:}\quad \frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big) f_j \lambda_j$$

Both saturate, but by different mechanisms, and that is the crux.

F&B saturates through the **sigmoid**, on the *difference* of summed influences. Adding a second defender to a zone already dominated still moves $PC$ — $\sigma(2) = 0.88$ against $\sigma(1) = 0.73$.

Spearman saturates through the **shared bracket**, on *total* control. Once $\sum_k PPCF_k \to 1$, every remaining player's contribution is multiplied by approximately zero. The second defender adds almost nothing.

**Prediction:** F&B assigns more extreme control values in crowded areas than Spearman does. Disagreement should scale with **local player density**, and therefore be worst in the penalty area and around the ball — the regions that dominate valuation.

### 2. Attack/defence asymmetry — the cleanest to measure

Spearman fits $\kappa = 1.72$, scaling defenders' control rate. F&B sets $\lambda_1 = \lambda_2 = 1$.

**Prediction:** a systematic shift of Spearman's surfaces toward defensive control, roughly uniform across the pitch. This is a **bias**, not noise, and should be visible as a non-zero mean difference before any correlation is computed. It is also the one difference that could be removed by refitting rather than by redesign.

### 3. Offside — sharp and localised

Spearman sets $\lambda_i = 0$ for attackers in offside positions. F&B has no offside term.

**Prediction:** disagreement concentrated in a **band beyond the last defender**, near-zero elsewhere. Easy to isolate, and the one place where the two models are not approximating the same quantity at all — one is answering a question about the laws of the game.

### 4. Ball travel time — direction unclear

Spearman holds $PPCF_j = 0$ until the ball could physically arrive, with flight simulated under aerodynamic drag. F&B treats arrival as instantaneous.

Both effects push the same way qualitatively — distant locations are harder to control — but by different mechanisms: a *time budget* in one, *influence decay with distance from the player* in the other. Whether they agree at distance depends on parameter values, and I cannot predict the sign without computing it. This is the difference least amenable to analysis.

## The Composite Prediction

If the above is right, the surfaces should agree **least where valuation depends on them most**: crowded areas near goal, and the final third around the offside line. A global correlation would therefore *understate* the practical disagreement, because most of the pitch is empty and both models will agree that empty space near one team is controlled by that team.

**A single global $r$ is the wrong summary.** The comparison should be stratified.

## Proposed Test

Compute both surfaces on the same tracking frames — a few hundred frames suffice.

1. **Global agreement.** Pearson $r$ over all cells, all frames. Expect high, and expect it to be misleading.
2. **Stratify by local player density** (players within, say, 10 m of the cell). Prediction 1 says disagreement rises monotonically with density.
3. **Mean signed difference**, globally. Prediction 2 says Spearman is systematically more defensive. Then **refit $\kappa = 1$** and re-measure: if the shift vanishes, the asymmetry is a parameter choice rather than a structural difference.
4. **Mask the offside band** and re-run. Prediction 3 says disagreement in that band is large and elsewhere unaffected.
5. **Stratify by distance from the ball**, to characterise the travel-time effect empirically since it resists analysis.
6. **The decisive step:** recompute [[obso|OBSO]] with the Gaussian surface substituted for PPCF, and compare player rankings. Surface disagreement only matters if it changes conclusions. If OBSO rankings are stable under substitution, the whole question is academic; if they are not, every cross-framework comparison in the vault needs a caveat.

Step 6 is the one that answers the question that actually matters, and it is the reason the earlier steps are worth doing — they diagnose *why* any ranking difference arises.

## What Would Change Depending on the Answer

**If they agree closely and rankings are stable** — the traditions are notational variants, and [[action-valuation-frameworks-compared|the synthesis]] can drop the caveat. The cheaper Gaussian model becomes the default with no cost.

**If they disagree structurally but rankings are stable** — downstream metrics are robust to their substrate, which is a reassuring and non-obvious finding worth stating.

**If rankings change** — then differences between [[obso|OBSO]], [[c-obso]] and the EPV surfaces are partly artefacts of control modelling, and the vault's comparative claims about them need qualifying. This is the outcome that would matter most and is not implausible, given that the predicted disagreement is concentrated exactly where those metrics take their largest values.

## Why Nobody Has Done It

Both models are published with enough detail to reimplement, and Spearman's parameters are stated. The obstacle is not difficulty — it is that **the two traditions have no overlapping authors and cite each other only in passing**. Fernández, Bornn & Cervone cite Spearman as related work on off-ball valuation; the [[keisuke-fujii|Fujii group]] adopts PPCF wholesale without comparing alternatives. Nobody has an incentive to test whether their own substrate is the right one.

That is the general shape of the [[action-valuation-frameworks-compared|benchmarking gap]] in this literature, visible here at the level of a component rather than a framework.

## See Also

- [[pitch-control]] · [[obso]] · [[c-obso]] · [[xsot]] · [[probability-surface]]
- [[expected-possession-value]] · [[off-ball-value]] · [[model-selection]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Fernández et al. Summary]]
- [[action-valuation-frameworks-compared]]
