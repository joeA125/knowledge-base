---
title: "Knowledge Base Overview"
type: synthesis
tags: []
sources: []
confidence: 0.4
provenance:
  extracted: 0
  inferred: 100
  ambiguous: 0
lifecycle: draft
created: 2026-04-22
updated: 2026-07-07
---

# Knowledge Base Overview

This page provides a high-level map of the knowledge contained in this wiki. It is rewritten periodically as the wiki grows. See [[index]] for the full page catalog.

## Domains Covered

- **Transformers & attention** — [[transformer]], [[attention-mechanism]], [[multi-head-attention]], [[scaled-dot-product-attention]], [[additive-attention]], [[positional-encoding]], [[encoder-decoder]], [[feed-forward-network]].
- **Recurrent networks & regularization** — [[lstm]], [[gated-recurrent-unit]], [[bidirectional-rnn]], [[recurrence]], [[dropout]], [[dropout-for-rnns]], [[label-smoothing]], [[regularization]].
- **Deep-network training** — [[residual-connections]], [[batch-normalization]], [[layer-normalization]], [[convolution]], [[dilated-convolution]], [[adam-optimizer]], [[scaling-laws]].
- **Generative models** — [[variational-autoencoder]], [[variational-lossy-autoencoder]], [[conditional-gan]], [[autoregressive-model]].
- **Bayesian rating & inference** — [[trueskill]], [[elo-rating-system]], [[glicko-rating-system]], [[factor-graph]], [[approximate-message-passing]], [[expectation-propagation]], [[gaussian-density-filtering]], [[bayesian-inference]], [[bayes-theorem]].
- **Computer vision for sport** — [[game-state-reconstruction]], [[camera-calibration]], [[homography]], [[optical-flow]], [[image-alignment]], [[object-detection]], [[feature-pyramid-network]], [[siamese-network]], [[semantic-segmentation]].
- **LLM reasoning, prompting & retrieval** — [[chain-of-thought]], [[react]], [[rlhf]], [[retrieval-augmented-generation]], [[scaling-laws]].

## Key Themes

- Attention displaced recurrence and convolution as the default sequence-modelling primitive; the [[transformer]] is the hub most other language-model pages connect back to.
- Regularisation and normalisation recur across nearly every architecture as the enablers of training depth and scale.
- A substantial applied cluster covers computer vision for football/broadcast analysis, from detection through camera calibration to full game-state reconstruction.

## Dashboards

Live Dataview-powered views:

- [[health|Wiki Health Dashboard]] — stale, low-confidence, draft, and orphan-risk pages.
- [[reinforcement|Reinforcement Dashboard]] — aging, single-source, and confidence-decay watch.
- [[sources|Source Tracking Dashboard]] — all raw sources and how often each is referenced.

## Open Questions

- `bert` and `gpt` are referenced as downstream impact but lack ingested primary sources (currently stubs).

## Growth Notes

- Wiki created: 2026-04-22
- Source summaries filed: ~29
- Concept + entity pages: ~90
