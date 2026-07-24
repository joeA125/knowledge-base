---
title: "Modeling Matches as Language: A Generative Transformer Approach for Counterfactual Player Valuation in Football — Source Summary"
type: source_summary
tags: [sports-analytics, counterfactual, transformer, event-prediction, player-evaluation, tokenization, constrained-decoding, multi-task-learning, event-stream-data, generative-model]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Modeling Matches as Language: A Generative Transformer Approach for Counterfactual Player Valuation

**Authors:** [[miru-hong]], [[minho-lee]], Geonhee Jo, Hyeokje Jo, [[pascal-bauer]], [[sang-ki-ko]]
**Affiliations:** University of Seoul; Saarland University; Deutscher Fussball-Bund (DFB)
**Published:** 2026

## Key Contribution

[[scoutgpt]] — a nanoGPT-based decoder-only [[transformer]] that treats football matches as token sequences and generates event streams **conditioned on a specific lineup**. This enables [[counterfactual-simulation|counterfactual simulation]]: replace one player in the context, re-generate the possession, and measure how the value changes.

## The Gap: Valuation Cannot Answer Transfer Questions

The framing critique is precise, and it applies to everything else in this vault's football cluster:

> Traditional valuation frameworks such as Expected Threat (xT) and VAEP quantify the value of observed events, but they do not generate how action sequences would evolve under a new tactical context.

A transfer is not a like-for-like substitution. Moving a player changes the tactical configuration and reshapes interaction patterns, so the question is how they will behave **under distribution shift** — which extrapolating past performance cannot answer.

The forecasting models ([[seq2event]], [[nmstpp]], [[sig-model]]) get closer but are built to predict *observed* continuations, and mostly lack the entity-conditioning needed to hold context fixed while swapping one player.

## Method

### Tokenization
Each event is a 10-dimensional tuple (team, position, player, action, start/end coordinates, time delta, outcome) flattened into **10 atomic tokens**, with continuous variables quantised into bins. A **56-token context block** encodes both lineups (11 position/player pairs per team) plus period and score. See [[tokenization]].

Goal-scored and goal-conceded flags are deliberately **removed from the input** to prevent label leakage during autoregressive generation, while being retained as supervision targets.

### Multi-task objective
Standard next-token cross-entropy plus two auxiliary heads predicting goal-scored and goal-conceded probabilities, activated **only at outcome-token positions**:

$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{gen}} + \mathcal{L}_{\text{aux}}$$

This aligns generated sequences with match value ([[vaep]]) rather than mere likelihood — a plain next-token objective would favour frequent actions regardless of tactical consequence. See [[multi-task-learning]].

### Constrained decoding
Unconstrained sampling produces syntactically valid but physically impossible sequences. ScoutGPT applies **state-dependent logit masking**: $P(s_t \mid s_{<t}) \propto \text{softmax}(z_t + M_t)$ with $M_t(k) = -\infty$ for invalid tokens. Rules come from the VERSA transition validator plus local consistency constraints (no immediate self-reception, no defensive actions while possession is retained). See [[constrained-decoding]].

Player identity is **not generated** — it is resolved deterministically from the context lineup given the predicted team and position, breaking ties by proximity to the player's reference location. This is what makes the counterfactual clean: swapping the lineup swaps the player without the model being free to overrule it.

## Data

K League 1 and 2, 2021–2025: 2,283 matches, 222,940 episodes, 1,490 players, 27.5 events per episode. Chronological split — 2021–23 train, 2024 validation, 2025 test.

An *episode* is defined between ball-out-of-play instances, a different segmentation from the possession-based units used by [[sig-model]] and [[football-event-sequences-point-process-mixture|Amezouwui et al.]]

## Results

### Value prediction (within 15 seconds)
| Method | GS AUC | GS Brier | GC AUC | GC Brier |
|---|---|---|---|---|
| ScoutGPT auxiliary head | 0.8344 | **0.0069** | **0.8153** | **0.0016** |
| CatBoost | **0.8424** | 0.0075 | 0.8051 | 0.0021 |

Notably **[[gradient-boosting|CatBoost]] still wins on goal-scored discrimination and calibration.** The generative model is better on conceding risk and on Brier score, but this is not a clean sweep — a specialised discriminative model remains competitive at the narrow task it was built for.

### Event modelling
ScoutGPT beats the LEM Transformer baseline (a [[nmstpp]]-derived architecture) across most attributes, with the largest gains on continuous variables: start-$x$ MAE 4.59 → 0.97, time MAE 1.42 → 0.75.

### Ablations
Removing lineup information costs 9 points of position accuracy — the model genuinely uses who is on the pitch. Removing context costs most on timing ($R^2$ −0.18). A few isolated metrics *improve* under ablation, which the authors read as simplified variants fitting specific marginals better while losing tactical coherence.

## Player Embeddings Learn Position Without Supervision

Position tokens are masked during training, forcing the model to infer role from player identity plus event context. The resulting embeddings, projected by t-SNE, **separate by position anyway** — and the geometry is tactically sensible: defensive midfielders sit between centre-backs and attacking midfielders; full-backs separate vertically.

Players at cluster boundaries (Jinsub Park between CDM and CB, Seungwon Jeong between full-back and midfield) turn out to be genuinely versatile, having played multiple roles — Park logged 4,383 minutes at CB, 2,081 at CDM, 1,849 at CM.

Masking helps quantitatively too: same-player cross-season retrieval improves to 9.20/21.97/30.90 (Top-1/5/10) from 8.48/20.02/27.05 without masking. Removing the shortcut produces better representations.

## Transfer Simulation

Across the top 40 transferred players by post-transfer minutes, simulated season VAEP beats a naive carry-over projection: **MAE 1.25 vs 1.88**.

The illustrative case is Jinsu Kim, whose naive projection (7.00) badly underestimated his actual post-transfer contribution (11.07); ScoutGPT predicted 11.63, an error of 0.56 against the naive 4.07. His move was widely regarded as unexpectedly successful, and he was made captain the following season.

### Context intervention
Holding the player fixed and varying match minute and score state produces coherent shifts: negative VAEP deltas at minute 0 across all score states (cautious early play, most pronounced when drawing), positive at minute 40 (more proactive late, strongest when trailing). The model has learned game-state-dependent behaviour, not just player-dependent behaviour.

### Simulation stability
Self-to-self reconstruction error decreases monotonically with Monte Carlo sample count (per-episode mean $1.9 \to 1.5 \times 10^{-3}$ from 1 to 20 samples), so counterfactual estimates need averaging over samples rather than single rollouts.

## Limitations

- Single league (K League), five seasons — generalisation untested.
- Only 40 transfers evaluated, all within the same league.
- On-ball events only; the authors identify tracking-based off-ball signals as future work.
- Generation is episode-level, not full-match.
- The naive baseline is weak (previous-season VAEP adjusted only for minutes), so the comparison flatters the model somewhat.

## See Also

- [[scoutgpt]]
- [[counterfactual-simulation]]
- [[large-event-model]]
- [[tokenization]]
- [[constrained-decoding]]
- [[multi-task-learning]]
- [[vaep]]
