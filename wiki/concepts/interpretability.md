---
title: "Interpretability"
type: concept
tags: [interpretability, machine-learning, evaluation, feature-attribution, model-decomposition, sports-analytics, deep-learning]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/expected_value_possession_framework.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Interpretability

The degree to which a model's outputs can be explained in human terms. Referenced throughout this vault as a design axis — usually as the thing traded away for predictive richness — and worth a page because the football sources turn out to mean at least **four different things** by it.

## Four Distinct Senses

| Sense | Question answered | Example in vault |
|---|---|---|
| **Structural simplicity** | Can I follow the whole model? | [[expected-threat\|xT]] — a zone transition matrix |
| **Attribution** | Which inputs moved this prediction? | [[shap]] on [[vdep]] |
| **Compositional** | Which sub-quantities make up this number? | [[structured-model-decomposition\|Fernández et al.]] |
| **Behavioural guarantee** | Can I trust what the output's movements mean? | [[martingale-epv\|Martingale EPV]] |

These are not degrees of one property. A model can be strong on one and absent on another, and conflating them causes real confusion about what any given framework offers.

## The Trade-Off, and Two Escapes From It

The vault's [[action-valuation-frameworks-compared|valuation comparison]] shows a consistent pattern: richer state representations buy sensitivity and pay in interpretability. xT is legible because it is simple; [[vaep]] and [[martingale-epv]] are opaque because they are not.

Two sources attack the trade-off rather than accepting a position on it, and they do so differently.

**Compositional escape.** [[expected-value-possession-framework|Fernández, Bornn & Cervone]] keep the rich model and split it along axes a coach already uses — pass / drive / shoot, success / failure, and where. The parts are individually inspectable even though no part is simple. Crucially the split must follow *domain* structure: a mathematically valid decomposition along axes nobody thinks in buys nothing.

**Guarantee escape.** [[martingale-epv|Cervone et al.]] make the model no simpler but prove a property of it — stochastic consistency — so the value curve's movements are known to reflect events rather than estimator noise. Interpretability here is not "I can see inside" but "I know what the outside means".

The same authors chose differently five years apart, which is the sharpest available illustration that these are genuinely distinct routes rather than degrees. See [[martingale-epv]].

## What Interpretability Is For

Worth being explicit, because the answer changes what counts as sufficient.

**For the modeller** — validating that a model learned something sensible. [[shap]] serves this: Toda et al. use it to confirm [[vdep]]'s off-ball features carry the signal rather than padding the vector.

**For the domain user** — acting on the output. A coach needs to know *what to change*, and neither a feature-attribution plot nor a simple transition matrix supplies that directly. This is what the compositional route targets, and why its applications are visual and situational.

**For accountability** — justifying a decision to someone affected by it. Barely engaged with in this vault, but it is the sense that dominates interpretability research elsewhere, and the standards are much higher.

A framework can be excellent for the first and useless for the second. [[expected-threat|xT]] is highly legible yet says nothing about what a player should have done; VDEP's SHAP plot reassures a modeller and would not survive a team meeting.

## Interpretability Is Not Validity

The most important caveat, and one the vault's sources handle unevenly.

An interpretable model can be confidently wrong. SHAP attributions explain what the *model* used, not what drives the world — a model leaning on a confounded feature produces a clean, plausible-looking plot. Likewise a decomposition into legible parts is legible whether or not the parts are correct; what licenses trust in [[expected-value-possession-framework|Fernández et al.'s]] recombination is the per-component [[probability-calibration|calibration]], not the decomposition itself.

The risk is sharpest where output is **visually persuasive**. Heatmaps on a pitch read as evidence in a way a correlation table does not — see the validation warning on [[tactical-analysis]]. Interpretability increases the *confidence* a claim commands, and does nothing to increase how much it deserves.

## Elsewhere

The distinctions here map onto the wider literature: intrinsically interpretable models (linear, rule-based, shallow trees) against post-hoc explanation of black boxes (SHAP, LIME, saliency), and the standing argument that for high-stakes decisions the former should be preferred rather than the latter retrofitted. The football case is a useful test of that argument, since the most legible framework (xT) is also the most [[split-half-reliability|reliable]] — but the most useful to a coach is neither.

## See Also

- [[shap]] · [[structured-model-decomposition]] · [[feature-engineering]]
- [[martingale-epv]] · [[expected-threat]] · [[vaep]] · [[vdep]]
- [[probability-calibration]] · [[tactical-analysis]] · [[split-half-reliability]]
- [[action-valuation-frameworks-compared]]
