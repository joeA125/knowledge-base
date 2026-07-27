---
title: "Large Event Model (LEM)"
type: concept
tags: [sports-analytics, generative-model, transformer, event-prediction, event-stream-data, tokenization, sequence-modelling, pre-training, time-series]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-27
---

# Large Event Model (LEM)

A Large Event Model ([[tiago-mendes-neves|Mendes-Neves]], [[luis-meireles|Meireles]] & [[joao-mendes-moreira|Mendes-Moreira]], 2024) applies the large-language-model recipe to sports event streams: decompose each event into attributes, tokenise, and train a [[transformer]] on next-token prediction over whole matches. The ambition is a **foundation model for football** — one generative model supporting many downstream tasks rather than a bespoke model per task.

> **Provenance note:** vault knowledge of the LEM papers themselves comes from citations in [[scoutgpt-counterfactual-player-valuation|Hong et al. (2026)]], not the primary sources. The group's *earlier* work is held directly — see below.

## The Group's Earlier Work

The same three authors produced [[football-performance-time-series|"Valuing Players Over Time"]] before the LEM papers, and the contrast is instructive about what the foundation-model turn was reacting against.

| | Valuing Players Over Time | LEM |
|---|---|---|
| Model | [[random-forest\|Random Forest]] regressor | [[transformer]] decoder |
| Target | Hand-designed time-decayed value label | Next token |
| Pipeline | Value actions → aggregate per game → [[smoothing\|smooth]] → analyse | Learn the event distribution directly |
| Contribution | The **aggregation** ([[player-rating-time-series\|time-series framing]]) | The **representation** |

The earlier paper takes a valuation model as given and extracts more from it by better aggregation. The LEM papers discard the hand-built pipeline entirely. What persists is the conviction that football event streams are **sequential data with exploitable structure**, argued first at the level of match-by-match ratings and later at the level of individual events.

Notably, the earlier paper's derived quantities — [[performance-volatility|volatility]], [[player-development-curve|development curves]] — have **no counterpart in the LEM line**. A generative event model produces distributions over what happens next; it does not by itself produce a longitudinal account of a career. These remain complementary rather than superseded.

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

The same group's earlier Random-Forest-based work is a further data point for the sceptical reading: its most durable contributions came from **how ratings were aggregated**, not from model capacity.

## See Also

- [[scoutgpt]]
- [[seq2event]]
- [[nmstpp]]
- [[sig-model]]
- [[tokenization]]
- [[constrained-decoding]]
- [[player-rating-time-series]]
- [[tiago-mendes-neves]] · [[luis-meireles]] · [[joao-mendes-moreira]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
- [[football-performance-time-series|Valuing Players Over Time Summary]]
