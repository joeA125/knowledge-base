---
title: "Do the two pitch-control traditions agree?"
type: question
tags: [pitch-control, sports-analytics, optical-tracking-data, probability-surface, evaluation, model-selection, needs-review]
sources: [raw/papers/physics_based_pass_probabilities.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.75
provenance:
  extracted: 55%
  inferred: 40%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Do the two pitch-control traditions agree?

**Status:** Open, and now **fully specified from primary sources on both sides.** Both origin papers are held. Analytically partial; empirically untested.

## Why They Never Engaged

> **Added 2026-07-27** on ingest of [[wide-open-spaces-space-creation|Fernández & Bornn (2018)]], the second origin.

Neither tradition cites the other. Both position against **[[voronoi-tessellation]]** — discrete dominance regions, the field's prior default — as do all their cited predecessors.

**They are siblings framed against a common ancestor, not rivals.**^[generated: the inference that a shared opponent explains the mutual silence is drawn here; neither source states it. rests-on: source:fb-voronoi-critique, absence:no-held-source-compares-ppcf-and-gaussian]

That reframes the absence. Two methods that **never engaged** is a different situation from two that engaged and disagreed: the silence is a fact about the field's structure, not evidence about relative merit. Neither paper had a reason built into it to compare against the other, because each was written as an improvement on Voronoi rather than as a competitor to a contemporary.

It also means the comparison is **nobody's responsibility**, which is why it has stayed undone for eight years.

## They Are Not Answering the Same Question

**PPCF originates as a pass-reception model.** [[physics-based-pass-probabilities|Spearman et al. (2017)]] ask *who will receive this pass*, fit parameters to that outcome, and derive a control surface by evaluating the same model for an imaginary stationary ball everywhere. Control is a **by-product of reception**.

**The Gaussian model originates as a spatial-dominance model.** [[wide-open-spaces-space-creation|Fernández & Bornn (2018)]] ask *who owns this region*, with no reception event in view, in order to support [[space-occupation-gain|space-creation metrics]].

Related but distinct questions. A location may be one a team would reliably receive a pass at and not one they "control" territorially, or the reverse. **Any disagreement is therefore partly definitional** — a reason to expect divergence, and a reason not to read it as one being wrong.

## The Validation Asymmetry, Correctly Stated

> **Superseded, 2026-07-27.** This page previously recorded the Gaussian tradition as "validated against nothing directly", implying a rigour gap. That was too strong.

| | PPCF | Gaussian influence |
|---|---|---|
| Parameters | **MLE-fitted**, stat and syst errors | **Expert-set** by FC Barcelona analysts |
| Validated against | **5,471 held-out pass receivers** — 81% team, 68% player | **Expert video review**, two analysts |
| Ground truth exists? | **Yes** — who received the pass is a fact | **No**, stated explicitly by the authors |

Fernández & Bornn say plainly that *"there is no existance of ground truth data regarding the quantification of spaces in soccer"*, and validate accordingly.

**So the asymmetry is in what is available, not in what was attempted.** One target is directly observable and was tested against outcomes; the other is not observable at all and was tested against expert judgement. Both did what their target permitted.

That still has a consequence for this question: **the two are not equally warranted as models of pass reception**, because only one was ever checked against it. If they diverge on a reception-flavoured question, the prior should favour PPCF — not because it is better work, but because it is the only one tested on that quantity.

## What Can Be Settled Analytically

Four structural differences; three yield directional predictions.

### 1. Saturation under crowding — the largest expected effect

$$\text{F\&B:}\quad PC = \sigma\Big(\textstyle\sum_{att} I_i - \sum_{def} I_j\Big) \qquad \text{Spearman:}\quad \frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big) f_j \lambda_j$$

Both saturate, by different mechanisms — and **the F&B behaviour is design intent, stated by its authors**:

> *"a single player without any influence of any other player at its current location only controls $\text{logistic}(1) = 0.73$ of the space. This provides the need of higher density of players near a given area to provide higher level of control in that area."*

So a lone player does **not** own his own location, and density is required for high control. Spearman saturates on *total* control instead: once $\sum_k PPCF_k \to 1$, every remaining contribution is multiplied by approximately zero, and a lone uncontested player approaches full control.

**Prediction:** the two diverge most where player density is high — and in a specific direction, since F&B suppresses lone-player control and rewards clustering while PPCF does neither. Disagreement should scale with **local player density**, worst in the penalty area and around the ball.

### 2. Attack/defence asymmetry — the cleanest to measure

[[obso|Spearman (2018)]] fits $\kappa = 1.72$ for defenders; the 2017 origin has **no such term**, so this is a refinement of the tradition rather than a defining feature. F&B has none.

**Prediction:** a systematic shift toward defensive control in the 2018 formulation only, roughly uniform — a **bias** visible as a non-zero mean difference before any correlation, and removable by refitting.

### 3. Offside — sharp and localised

Spearman (2018) zeroes attackers in offside positions; F&B has no offside term.

**Prediction:** disagreement concentrated in a **band beyond the last defender**, near-zero elsewhere. The one place the two are not approximating the same quantity at all.

### 4. Distance to the ball — same intent, opposite mechanism

Both encode it, differently. PPCF models **ball flight time** with drag, so a distant location is harder to reach for both teams. F&B scales the **influence radius** by distance to the ball — 4 m on the ball, 10 m at 20 m away — so a distant player has *wider* influence.

Direction of disagreement is not analytically predictable; the two mechanisms are not monotone transformations of each other.

## The Composite Prediction

The surfaces should agree **least where valuation depends on them most**: crowded areas near goal, and the final third around the offside line. A global correlation would therefore *understate* practical disagreement, because most of the pitch is empty and both models agree that empty space near one team belongs to that team.

**A single global $r$ is the wrong summary.** Stratify.

## Proposed Test

Compute both on the same tracking frames — a few hundred suffice.

1. **Global agreement.** Pearson $r$ over all cells. Expect high, and expect it to mislead.
2. **Stratify by local player density.** Prediction 1 says disagreement rises monotonically, and that F&B is the lower of the two for isolated players.
3. **Mean signed difference**, then refit with $\kappa = 1$. If the shift vanishes, the asymmetry is a parameter choice rather than a structural difference.
4. **Mask the offside band** and re-run.
5. **Stratify by distance from the ball**, to characterise difference 4 empirically since it resists analysis.
6. **The reception check.** Evaluate the Gaussian surface against **actual pass receivers**, as the 2017 paper does for PPCF. This puts both traditions on one directly observable criterion for the first time, and the 2017 paper supplies the protocol. Note this tests the Gaussian model on a question it was not built for — informative about substitutability, not about whether it succeeds on its own terms.
7. **The decisive step.** Recompute [[obso|OBSO]] with the Gaussian surface substituted for PPCF and compare player rankings. Surface disagreement only matters if it changes conclusions.

Steps 6 and 7 answer different things and both are worth running: 6 asks whether the surfaces are interchangeable *as models*, 7 whether the choice matters *in practice*.

## What Would Change Depending on the Answer

**Agree closely, rankings stable** — notational variants. The cheaper model becomes the default at no cost.

**Disagree structurally, rankings stable** — downstream metrics are robust to their substrate. Reassuring and non-obvious.

**The Gaussian model predicts reception poorly** — then using it as a control term inside a value model is doing something different from what PPCF does there, and the two are not substitutable even where they correlate.

**Rankings change** — differences between [[obso|OBSO]], [[c-obso]], [[drso]], [[space-occupation-gain|SOG]] and the EPV surfaces are partly artefacts of control modelling, and the vault's comparative claims about them need qualifying. Not implausible: predicted disagreement concentrates exactly where those metrics take their largest values.

**If this last case holds, it would also justify a pitch-control synthesis** — a result spanning six sources with consequences for five frameworks. Absent a result, the material sits correctly distributed across [[pitch-control]], this page, and [[voronoi-tessellation]].

## See Also

- [[pitch-control]] · [[voronoi-tessellation]] · [[pass-probability-model]] · [[pitch-value-model]] · [[probability-surface]]
- [[obso]] · [[c-obso]] · [[drso]] · [[xsot]] · [[space-occupation-gain]] · [[off-ball-value]]
- [[model-selection]] · [[shot-value-formulations-compared]] · [[expected-possession-value]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]]
- [[physics-based-pass-probabilities|Spearman 2017]] · [[wide-open-spaces-space-creation|Fernández & Bornn 2018]] · [[beyond-expected-goals|Spearman 2018]]
