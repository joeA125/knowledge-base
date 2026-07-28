---
title: "Event Prediction"
type: concept
tags: [event-prediction, sequence-modelling, sports-analytics, point-process, machine-learning, evaluation, generative-model, action-valuation]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Event Prediction

Forecasting the next event in a sequence — in football, what action happens next, where, and when. One of the [[action-valuation-frameworks-compared|five distinct tasks]] the vault's football sources address, and the one whose relationship to valuation is most often misread.

## What Is Being Predicted

An event has several attributes, and frameworks differ in which they commit to:

| | Type | Location | Time | Actor |
|---|---|---|---|---|
| [[seq2event]] | Yes | Zone | No | No |
| [[nmstpp]] | Yes | Zone (20) | **Yes** | No |
| [[sig-model]] | Yes | **Exact $(x,y)$** | No | No |
| [[scoutgpt]] | Yes | Binned | Yes | **Conditioned on lineup** |

Each addition is a criticism of the last: NMSTPP holds that ignoring *when* discards the difference between a quick counter and a slow build-up; Sig-Model that zone discretisation throws away the geometry that matters; ScoutGPT that none can be conditioned on a hypothetical lineup.

Distinct from [[trajectory-prediction]], which forecasts continuous *positions* of all agents rather than discrete events. The two are converging — both now supply inputs to valuation — but the output type differs.

## Prediction as a Route to Valuation

The most consequential thing about this task is that **it produces metrics as a by-product**, and those metrics have a property the [[action-valuation|valuation]] frameworks lack.

| Forecasting model | Derived metric |
|---|---|
| [[seq2event]] | poss-util |
| [[nmstpp]] | [[hpus]] |
| [[sig-model]] | [[lpv]] |
| GVRNN | [[c-obso]] |

The property: **a forecasting model needs no outcome labels.** Every event is a training example, so the goal-sparsity problem that cripples direct valuation never arises. [[hpus]] is the cleanest demonstration — it uses **no goal or shot-outcome data at any stage** yet correlates 0.92 with season xG and −0.78 with league position, against xG's −0.81.

That is the same escape route [[rare-event-proxy-targets|proxy targets]] take, arrived at from a different direction: rather than swap a rare target for a frequent one, dispense with the target entirely and read value out of the *dynamics*.

The cost is that value becomes an interpretation of the forecast rather than a modelled quantity. HPUS is a formula applied to NMSTPP's outputs; a different formula over the same forecasts would give a different metric, and nothing adjudicates.

## Modelling Approaches

**[[point-process|Marked spatio-temporal point processes]]** treat events as arrivals in continuous time with attached marks. Natural for the *when* question, and the framing behind [[nmstpp]] and the [[football-event-sequences-point-process-mixture|possession mixture model]]. See [[neural-temporal-point-process]].

**Sequence models** treat a match as a token stream and apply language-model machinery — [[transformer]] encoders in Seq2Event and NMSTPP, a GPT-2 decoder in [[scoutgpt]]. See [[large-event-model]] for the football-as-language framing, and [[tokenization]] and [[constrained-decoding]] for the representational choices it forces.

**Representation-first approaches** change the input rather than the architecture. [[sig-model]] encodes the whole possession as a [[path-signature]] and uses a plain feed-forward network.

## The Feature-Engineering Finding

Worth isolating because it generalises beyond sport. Seq2Event **degrades without** handcrafted geometric features; Sig-Model **degrades with** them.

The reading: engineered features are a crutch for a representation that cannot recover the geometry itself. Where the representation can — as a path signature can, by construction — adding them injects redundancy and noise. See [[feature-engineering]].

This sits in unresolved tension with the tracking-data line, where [[expected-value-possession-framework|Fernández et al.]] and [[vdep]] both hand-engineer extensively and defend it. The resolution offered is that they optimise different things — accuracy versus communicability — but neither side has tested the other's claim.

## Evaluation

Held-out likelihood, Brier score, and [[kl-divergence]] against empirical zone-conditioned distributions. Unlike valuation, this task has **genuine ground truth**: the next event is observed.

That advantage is easy to over-read. A model can forecast well and still yield a poor metric, because the step from forecast to value is unvalidated. The vault's [[predictive-validity]] results — where poss-util, HPUS and LPV are compared against next-match outcomes — test the *metrics*, not the forecasters, and are the closest thing to an end-to-end check.

## See Also

- [[nmstpp]] · [[sig-model]] · [[seq2event]] · [[scoutgpt]] · [[eventgpt]]
- [[hpus]] · [[lpv]] · [[trajectory-prediction]] · [[large-event-model]]
- [[point-process]] · [[neural-temporal-point-process]] · [[path-signature]] · [[transformer]]
- [[generative-model]] · [[rare-event-proxy-targets]] · [[feature-engineering]] · [[predictive-validity]]
- [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[transformer-point-process-football-event-modelling|NMSTPP Summary]] · [[understanding-football-possessions-path-signatures|Sig-Model Summary]]
