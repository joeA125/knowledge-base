---
title: "Probability Surface"
type: concept
tags: [probability-surface, spatiotemporal, sports-analytics, deep-learning, computer-vision, action-valuation, off-ball, interpretability]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Probability Surface

Predicting a value at **every** location on a spatial domain rather than only where something was observed. In football, a $104 \times 68$ grid over the pitch giving — for each cell — the probability a pass there succeeds, the probability it is chosen, or the [[expected-possession-value|EPV]] that would result.

## The Problem It Solves

Passes can go anywhere. Every prior model in this vault that values passing evaluates only the pass that *happened*, or restricts the option set to teammates' current positions (Power et al., [[martingale-epv|Cervone et al.]], Hubáček et al.).

That restriction quietly forecloses the question coaches actually ask. "Was that a good pass?" is answerable pointwise. **"Where should the ball have gone?"** is not — it requires a value at every alternative, including empty space a teammate could run into.

A surface answers both. The observed pass is one cell; the best available option is the maximum over the surface; the gap between them is the coaching insight. In the [[expected-value-possession-framework|worked example]] the realised EPV is 0.032 while the best available pass would have yielded 0.112.

## What Changes Conceptually

The shift is from valuing **actions** to valuing **options**, and it has consequences beyond convenience.

**Counterfactuals become cheap.** [[counterfactual-simulation]] normally requires a generative model and Monte Carlo rollouts. A surface gives the value of every alternative destination directly, with no generation — a much weaker counterfactual (it holds everything else fixed) but essentially free.

**[[off-ball-value|Off-ball value falls out]].** If the surface gives the EPV of a pass to every location, then the value of a player *standing* at a location is available immediately. This is how the framework produces the vault's first off-ball valuation without modelling off-ball behaviour explicitly.

**Aggregate analysis becomes spatial.** Summing positive EPV-added over locations across many possessions produces heatmaps of where a team creates and concedes advantage — the basis of the framework's [[tactical-analysis|pressing analysis]].

## Requirements

Three, and the middle one is the hard part:

1. **A spatially-structured architecture.** Predicting 7,072 correlated values needs weight sharing; see [[fully-convolutional-network]] and [[soccermap]].
2. **A way to train without full ground truth.** Only one cell per pass is ever observed. See [[single-pixel-supervision]].
3. **Constraints appropriate to the quantity.** A *selection* surface must sum to 1 (softmax); a *probability* surface needs each cell in $[0,1]$ (sigmoid); a *value* surface here needs $[-1, 1]$ (sigmoid plus linear transform).

The third is easy to get wrong. Pass success and pass selection look similar and are not: one is "would this work?", the other "would this be attempted?" — and only the latter is a distribution over the field.

## Calibration Across a Surface

A subtlety the source handles and most surface work does not. Calibration is normally checked on a set of predictions against their outcomes. For a surface, **only one cell per example ever has an outcome** — so calibration can only be verified along the thin diagonal of observed destinations.

The unobserved regions of the surface are therefore validated by *nothing directly*. Their plausibility rests on the smoothness the convolutional structure imposes and on the model getting the observed cells right. This is worth holding in mind before treating a whole surface as equally trustworthy: the far corner of the pitch is an extrapolation.

## Lineage

The idea is borrowed wholesale from dense prediction in computer vision — [[semantic-segmentation]], where the output is a per-pixel label rather than an image-level one. The architectural machinery ([[fully-convolutional-network|FCNs]], upsampling, multi-scale fusion, [[dilated-convolution|dilated convolutions]]) transfers directly; what does not transfer is the supervision, since segmentation datasets label every pixel and football does not.

## See Also

- [[soccermap]] · [[single-pixel-supervision]] · [[fully-convolutional-network]]
- [[expected-possession-value]] · [[off-ball-value]] · [[pitch-control]]
- [[semantic-segmentation]] · [[dilated-convolution]] · [[probability-calibration]]
- [[counterfactual-simulation]] · [[tactical-analysis]]
- [[expected-value-possession-framework|Source Summary]]
