---
title: "ScoutGPT"
type: concept
tags: [sports-analytics, counterfactual, transformer, event-prediction, player-evaluation, tokenization, constrained-decoding, multi-task-learning, generative-model]
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

# ScoutGPT

ScoutGPT ([[scoutgpt-counterfactual-player-valuation|Hong et al., 2026]]) is a nanoGPT-based decoder-only [[transformer]] that generates football event sequences conditioned on an explicit lineup, enabling [[counterfactual-simulation]] of player transfers.

## The Design Problem

To simulate "what if this player joined this team", a model must let you change **one** thing — the player — while holding tactical context fixed. Three design choices deliver that:

1. **Explicit lineup conditioning.** A 56-token context block encodes both starting elevens (position/player pairs) plus period and score, prepended to every episode.
2. **Player identity is never generated.** After predicting team and position, the player is resolved *deterministically* from the context lineup, breaking ties by proximity to the player's reference location. The model cannot overrule the intervention.
3. **Player-ID loss is excluded from training**, keeping the objective consistent with this inference procedure.

Without the second, autoregressive decoding could simply generate the player it expected, silently undoing the counterfactual.

## Architecture

| Stage | Detail |
|---|---|
| Preprocessing | VERSA validation — a formal state-transition model that corrects missing events and impossible orderings |
| Context | 56 tokens: 11 position/player pairs per team, period, score, cards |
| Event | 10 tokens each (team, position, player, action, start $x$/$y$, end $x$/$y$, $\Delta t$, outcome); continuous values binned 0–105 |
| Backbone | GPT-2 style pre-LayerNorm decoder blocks, causal MSA, GELU MLP |
| Heads | LM head plus two auxiliary heads for goal-scored and goal-conceded, active **only at outcome tokens** |
| Loss | $\mathcal{L}_{\text{gen}} + \mathcal{L}_{\text{aux}}$ — see [[multi-task-learning]] |
| Decoding | State-dependent logit masking — see [[constrained-decoding]] |

Goal flags are removed from the *input* to prevent label leakage while being kept as supervision targets — a small but essential detail, since a model that can see the goal label has nothing to predict.

## Results

**Value prediction.** The auxiliary heads beat [[gradient-boosting|CatBoost]] on goal-conceded AUC (0.8153 vs 0.8051) and on Brier score for both, but **lose on goal-scored AUC** (0.8344 vs 0.8424). Worth stating plainly: a specialised discriminative model remains competitive at the narrow task it was built for. The generative model's advantage is that it does other things too, not that it dominates.

**Event modelling.** Beats the LEM Transformer baseline (a [[nmstpp]]-derived architecture) on most attributes, with the largest gains on continuous variables — start-$x$ MAE 4.59 → 0.97, time MAE 1.42 → 0.75.

**Transfer prediction.** Across 40 transferred players, MAE 1.25 against a naive carry-over projection's 1.88. Though the baseline is weak — previous-season VAEP adjusted only for minutes — so the margin flatters the model.

## Emergent Player Representations

The most interesting finding is incidental to the main contribution. Position tokens are **masked during training**, forcing the model to infer role from player identity plus surrounding events rather than reading it off a label.

The learned player embeddings separate by position anyway, and the t-SNE geometry is tactically coherent: defensive midfielders sit *between* centre-backs and attacking midfielders, full-backs separate vertically by flank. Players who land at cluster boundaries turn out to be genuinely versatile — Jinsub Park, between CDM and CB, logged 4,383 minutes at CB, 2,081 at CDM and 1,849 at CM.

Masking also improves cross-season same-player retrieval (Top-1 9.20% vs 8.48% without). **Removing the shortcut produced better representations** — the same logic as [[masked-language-model|masked language modelling]], where hiding information forces richer context use.

## Context Sensitivity

Holding the player fixed and intervening on match state produces coherent behaviour: VAEP deltas are negative at minute 0 across all score states (cautious early play, most so when drawing) and positive at minute 40 (proactive late, most so when trailing). The model has learned game-state-dependent tactics, not only player-dependent ones.

## Relation to Other Football Models

| | Task | Conditioning |
|---|---|---|
| [[vaep]] / [[expected-threat\|xT]] | Value observed actions | None |
| [[seq2event]] / [[nmstpp]] / [[sig-model]] | Forecast next event | Game state |
| [[football-event-sequences-point-process-mixture\|Mixture model]] | Cluster possessions | None (latent) |
| **ScoutGPT** | **Simulate counterfactuals** | **Explicit lineup** |

ScoutGPT belongs to the [[large-event-model]] line — football as language, matches as token sequences — and is closest to EventGPT (Hong et al., 2025) by the same group, which generated only short fragments and so had to approximate the remaining value.

## Limitations

- Single league (K League), 5 seasons; 40 transfers, all intra-league.
- Episode-level generation, not full matches.
- On-ball events only.
- Causal validity is not established — see the caveats on [[counterfactual-simulation]].

## See Also

- [[counterfactual-simulation]]
- [[large-event-model]]
- [[tokenization]]
- [[constrained-decoding]]
- [[multi-task-learning]]
- [[gpt]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
