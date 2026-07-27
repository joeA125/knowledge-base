---
title: "SoccerMap"
type: concept
tags: [architecture, deep-learning, probability-surface, computer-vision, sports-analytics, optical-tracking-data, convolution]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# SoccerMap

A [[fully-convolutional-network|fully convolutional]] architecture (Fernández & Bornn, 2020) that maps a layered snapshot of tracking data to a full-pitch [[probability-surface|prediction surface]]. It is the feature-extraction block shared by all three pass models in the [[expected-value-possession-framework|EPV framework]].

## Input Representation

The game state is encoded as a stack of $104 \times 68$ layers, one per feature, each holding a value at every pitch location. For the pass success model there are 13:

- Possession-team player locations, and their $x$ and $y$ velocities
- Opponent locations, and their $x$ and $y$ velocities
- Distance to goal, distance to the ball
- Angle to goal; sine and cosine of the angle to the ball
- Sine and cosine of the angle between the ball carrier's velocity and each location

Player positions enter as **sparse layers** — a 1 at the player's cell, 0 elsewhere — while geometric quantities are dense. Stacking them means every layer is spatially registered to every other, so a convolution sees player presence, velocity and geometry at the same location simultaneously.

The pass EPV model uses 16 layers, adding contextual features ([[dynamic-pressure-lines]], outplayed players) and — notably — **the output of the pre-trained pass probability model as an input layer**. Surfaces feed surfaces.

## Architecture

Three resolution paths, fused:

1. At each of 1×, ½× and ¼× scale: two convolutional layers (32 then 64 filters, 5×5, stride 1) followed by max pooling to reach the next scale down.
2. Each scale produces its own prediction via 1×1 convolutions.
3. Coarse predictions are upsampled and merged into finer ones through **fusion layers** — 1×1 convolutions with linear activation.
4. A final 1×1 convolution emits the $104 \times 68 \times 1$ surface.

The multi-scale structure is the point. A 5×5 filter at full resolution sees roughly five metres of pitch; at ¼ scale it sees twenty. Local detail (is a defender adjacent?) and global structure (where is the defensive block?) both matter to whether a pass succeeds, and fusing scales lets one surface express both. The design is explicitly modelled on the "deep jet" hierarchy of Long et al.'s FCN work.

## What Makes It Reusable

The same architecture serves pass success probability, pass selection probability, and pass EPV — three different targets, three different output activations (sigmoid, softmax, sigmoid-plus-linear), one feature extractor.

Only the input layer stack and the loss change. That modularity is what makes the [[structured-model-decomposition|decomposed framework]] practical: each component gets a bespoke model without a bespoke architecture.

## Cost

Around 401,000 parameters for the surface models, against 128–231 for the shallow dense networks handling ball drives and shots — three orders of magnitude, for the three components that need spatial output.

Even so it predicts 899–984 examples per second, comfortably above the 10 Hz tracking rate. Real-time operation is what the framework's live "control room" application depends on, and it is a sharp contrast with [[martingale-epv|Cervone et al.'s]] 461-processor basketball model.

## Caveats

- **Fixed pitch discretisation.** $104 \times 68$ is roughly one cell per square metre. Sub-metre positioning is invisible, which may matter for tight-space passing.
- **Single-frame input.** The snapshot includes velocity, so first-order motion is represented, but the architecture has no memory — it cannot see that a run started three seconds ago.
- **Validated only where passes were observed.** See [[single-pixel-supervision]] and the calibration caveat on [[probability-surface]].

## See Also

- [[fully-convolutional-network]] · [[probability-surface]] · [[single-pixel-supervision]]
- [[expected-possession-value]] · [[structured-model-decomposition]]
- [[convolution]] · [[semantic-segmentation]] · [[feature-pyramid-network]] · [[dilated-convolution]]
- [[javier-fernandez]] · [[luke-bornn]]
- [[expected-value-possession-framework|Source Summary]]
