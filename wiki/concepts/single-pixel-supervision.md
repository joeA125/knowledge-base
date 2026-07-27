---
title: "Single-Pixel Supervision"
type: concept
tags: [weak-supervision, deep-learning, probability-surface, machine-learning, training-technique, computer-vision, sports-analytics]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.75
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Single-Pixel Supervision

Learning a full prediction surface when ground truth exists at exactly **one** location per training example. The loss is computed only at that location and back-propagated; the rest of the surface is never directly supervised.

## The Setup

A pass is attempted and arrives — or fails to arrive — at one destination. That is the only outcome the world provides. Yet the goal is a value at all $104 \times 68$ locations, including the thousands of passes that were never attempted.

The procedure is simple:

1. Predict the entire surface.
2. Select the cell corresponding to the observed destination.
3. Compute the loss between that single prediction and the observed outcome.
4. Back-propagate.

Formally, with $\hat{Y}$ the predicted surface and $d$ the observed destination, the loss is $\mathcal{L}(\hat{Y}_d, y)$ — everything else in $\hat{Y}$ contributes nothing to the gradient on this example.

## Why It Works

Nothing about the loss forces the unobserved cells toward anything. The surface is learned entirely through **parameter sharing**.

In a [[fully-convolutional-network|convolutional]] architecture the same filters are applied at every location. A gradient from one pixel updates weights that determine *all* pixels. So the network never learns "the value at cell (40, 22)"; it learns "the value at a cell with these local and global spatial properties" — and applies that function everywhere.

Aggregated over 288,619 training passes with destinations spread across the pitch, the shared function is constrained from many directions at once. The surface is an emergent consequence of a well-constrained local function, not something fit directly.

Two conditions make this viable, and both are worth stating because they bound where the trick transfers:

- **Translation-equivariant structure.** If value depended on absolute position in a way convolution cannot express, sharing would be wrong.
- **Coverage of the observation distribution.** Regions where passes are never attempted are extrapolation, not interpolation.

## Where It Sits in Weak Supervision

| Setting | Label available | Example |
|---|---|---|
| Full supervision | Every pixel | [[semantic-segmentation]] datasets |
| Image-level weak supervision | One label for the whole image | Pathak et al. (2015) |
| **Single-pixel** | One label at one known location | Pass surfaces |

The football case is unusual and in one respect *easier* than image-level weak supervision: the label is tied to a known location, so there is no ambiguity about which prediction it constrains. What is missing is not the correspondence but the coverage.

## The Selection Problem Underneath

Worth flagging because the source does not: **the observed destinations are not a random sample of the pitch.**

Players pass where passes are sensible. So the supervised cells concentrate in plausible passing regions, and the surface is best-constrained exactly where it is least surprising. In the improbable regions — the ones a coach might be most curious about, since "why did nobody try this?" is an interesting question — the surface is essentially the convolutional prior extrapolating.

This is a spatial instance of the structure on [[selection-bias]]: what gets observed depends on a judgement correlated with the quantity being modelled. It does not invalidate the approach, but it means confidence in a surface should not be treated as uniform across the pitch, and the reported [[probability-calibration|calibration]] figures — computed on observed destinations — do not speak to the unobserved regions at all.

## See Also

- [[probability-surface]] · [[soccermap]] · [[fully-convolutional-network]]
- [[semantic-segmentation]] · [[convolution]]
- [[selection-bias]] · [[probability-calibration]]
- [[expected-value-possession-framework|Source Summary]]
