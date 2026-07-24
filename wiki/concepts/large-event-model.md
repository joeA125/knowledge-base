---
title: "Large Event Model (LEM)"
type: concept
tags: [sports-analytics, generative-model, transformer, event-prediction, event-stream-data, tokenization, sequence-modelling, pre-training]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Large Event Model (LEM)

A Large Event Model (Mendes-Neves, Meireles & Mendes-Moreira, 2024) applies the large-language-model recipe to sports event streams: decompose each event into attributes, tokenise, and train a [[transformer]] on next-token prediction over whole matches. The ambition is a **foundation model for football** — one generative model supporting many downstream tasks rather than a bespoke model per task.

> **Provenance note:** vault knowledge comes from citations in [[scoutgpt-counterfactual-player-valuation|Hong et al. (2026)]], not the primary sources.

## The Football-as-Language Analogy

The analogy predates LEMs — [[seq2event]] framed its contribution as "learning the language of soccer" — and rests on a genuine structural parallel: both domains are sequences of discrete symbols with long-range dependencies, hierarchical structure, and meaning that depends on context.

What LEMs add is the **scale and generality** of the LLM playbook: train one autoregressive model on everything, then use it for whatever you need. Related efforts include RisingBaller ("a player is a token, a match is a sentence") and Baron et al.'s "foundation model for soccer".

## Where the Analogy Holds and Breaks

**Holds:**
- Discrete symbol sequences with context-dependent meaning.
- Long-range dependency — early build-up shapes later options.
- Next-token prediction as a self-supervised objective needing no labels.

**Breaks:**
- **Events are structured tuples, not atomic symbols.** A word is one token; a football event has actor, action, location, time, and outcome. Hence the multi-token-per-event schemes in [[tokenization]].
- **Hard validity constraints.** Ungrammatical text is merely odd; a physically impossible event sequence is wrong. This motivates [[constrained-decoding]], which has no real equivalent in open-ended text generation.
- **Value is not likelihood.** A plain next-token objective favours frequent actions — mostly passes — regardless of tactical consequence. [[scoutgpt]] adds explicit value supervision precisely because likelihood is the wrong target. See [[multi-task-learning]].
- **Data is scarce.** Text corpora run to trillions of tokens; the K League dataset behind ScoutGPT has 222,940 episodes across five seasons. [[scaling-laws]] suggest this constrains what the paradigm can reach.

## The Lineage in This Vault

| Model | Year | Contribution |
|---|---|---|
| [[seq2event]] | 2022 | Football as language; forecast location and action type |
| [[nmstpp]] | 2023 | Adds event *timing* via a [[point-process]] factorisation |
| LEM | 2024 | Full-match rollouts; foundation-model framing; 33 action types |
| [[sig-model]] | 2025 | Rejects fixed windows; [[path-signature]] encoding |
| [[scoutgpt]] | 2026 | Lineup conditioning enables [[counterfactual-simulation]] |

The consistent direction of travel is toward **richer conditioning and longer generation** — from predicting the next action type, to predicting when it happens, to generating whole possessions under hypothetical lineups.

## The Foundation-Model Question

Whether football has enough data for the foundation-model recipe to pay off remains open. The LLM analogy suggests scale is the lever, but a league season produces on the order of $10^5$ events against the $10^{12}$ tokens behind modern LLMs.

The vault's evidence is mixed. [[sig-model]] beats a transformer benchmark using a plain feedforward network on [[path-signature]] features, suggesting the right *representation* can substitute for scale in this regime. Conversely, ScoutGPT's lineup-conditioned generation does something no smaller model has demonstrated. The paradigms may be suited to different tasks rather than one superseding the other.

## See Also

- [[scoutgpt]]
- [[seq2event]]
- [[nmstpp]]
- [[sig-model]]
- [[tokenization]]
- [[constrained-decoding]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
