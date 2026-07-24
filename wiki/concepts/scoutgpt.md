---
title: "ScoutGPT"
type: concept
tags: [sports-analytics, counterfactual, transformer, event-prediction, player-evaluation, tokenization, constrained-decoding, multi-task-learning, generative-model, entity-embedding]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/eventgpt-player-impact-from-team-action-sequences.md]
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

ScoutGPT ([[scoutgpt-counterfactual-player-valuation|Hong et al., 2026]]) is a nanoGPT-based decoder-only [[transformer]] that generates football event sequences conditioned on an explicit lineup, enabling [[counterfactual-simulation]] of player transfers. It is the successor to [[eventgpt]] from the same group.

## The Design Problem

To simulate "what if this player joined this team", a model must let you change **one** thing while holding tactical context fixed. Three choices deliver that:

1. **Explicit lineup conditioning.** A 56-token context block encodes both starting elevens plus period and score, prepended to every episode.
2. **Player identity is never generated.** After predicting team and position, the player is resolved *deterministically* from the lineup, breaking ties by proximity to their reference location. The model cannot overrule the intervention. (Established in [[eventgpt]].)
3. **Player-ID loss is excluded from training**, keeping the objective consistent with this inference procedure.

## Architecture

| Stage | Detail |
|---|---|
| Preprocessing | VERSA validation — a formal state-transition model correcting missing events and impossible orderings |
| Context | 56 tokens: 11 position/player pairs per team, period, score, cards |
| Event | 10 tokens each; continuous values binned 0–105 |
| Backbone | GPT-2 style pre-LayerNorm decoder blocks, causal MSA, GELU MLP |
| Heads | LM head plus goal-scored and goal-conceded heads, active **only at outcome tokens** |
| Loss | $\mathcal{L}_{\text{gen}} + \mathcal{L}_{\text{aux}}$ — see [[multi-task-learning]] |
| Decoding | State-dependent logit masking — see [[constrained-decoding]] |

Goal flags are removed from the *input* to prevent label leakage while being kept as supervision targets.

## Results

**Value prediction.** The auxiliary heads beat [[gradient-boosting|CatBoost]] on goal-conceded AUC (0.8153 vs 0.8051) and on Brier for both, but **lose on goal-scored AUC** (0.8344 vs 0.8424). A specialised discriminative model remains competitive at the narrow task it was built for; the generative model's advantage is that it does other things too.

**Event modelling.** Beats the LEM Transformer baseline on most attributes, with the largest gains on continuous variables — start-$x$ MAE 4.59 → 0.97, time MAE 1.42 → 0.75.

**Transfer prediction.** MAE 1.25 across 40 transferred players against a naive carry-over projection's 1.88 — though that baseline is weak (previous-season VAEP adjusted only for minutes).

## What Changed from EventGPT

| | [[eventgpt]] (2025) | ScoutGPT (2026) |
|---|---|---|
| Event format | [[spadl]] | VERSA, with validity correction |
| Tokens per event | 7 | 10 |
| Value handling | [[on-ball-value\|rOBV]] as a **token** | Auxiliary **heads** at outcome positions |
| Decoding | Unconstrained | [[constrained-decoding\|Validity-masked]] |
| Counterfactual | Re-score the fixed observed sequence | **Re-generate** the sequence |
| League | Premier League | K League |

The third and fifth rows are the substantive changes. EventGPT's headline counterfactual holds the observed event sequence fixed and re-evaluates rOBV under a substituted player — estimating how that player would *value* the same situations. ScoutGPT regenerates the sequence, estimating how the player would *change what happens*. The second is the stronger counterfactual.

> **A note on characterisation.** ScoutGPT's related-work section describes EventGPT as generating "only short fragments of a sequence, requiring the remaining value to be approximated via residual OBV." Reading EventGPT directly, its transfer-simulation procedure primarily re-scores fixed sequences rather than generating fragments, though it does generate in places. The two descriptions are not straightforwardly reconcilable, and the successor's account of its predecessor should be treated as a characterisation rather than a neutral summary — a general caution when a group benchmarks against its own prior work.

## Emergent Player Representations

Position tokens are **masked during training**, forcing the model to infer role from identity plus surrounding events. The learned [[player-embedding|embeddings]] separate by position anyway, with tactically coherent geometry, and masking improves cross-season same-player retrieval (Top-1 9.20% vs 8.48%). Removing the shortcut produced better representations — the same logic as [[masked-language-model|masked language modelling]].

## Context Sensitivity

Holding the player fixed and intervening on match state produces coherent behaviour: VAEP deltas negative at minute 0 across all score states (cautious early play), positive at minute 40 (proactive late, most so when trailing). The model has learned game-state-dependent tactics, not only player-dependent ones.

## Relation to Other Football Models

| | Task | Conditioning |
|---|---|---|
| [[vaep]] / [[expected-threat\|xT]] | Value observed actions | None |
| [[seq2event]] / [[nmstpp]] / [[sig-model]] | Forecast next event | Game state |
| [[football-event-sequences-point-process-mixture\|Mixture model]] | Cluster possessions | None (latent) |
| [[eventgpt]] | Re-value under substitution | Player identity |
| **ScoutGPT** | **Simulate counterfactuals** | **Explicit lineup** |

Both belong to the [[large-event-model]] line — football as language, matches as token sequences.

## Limitations

- Single league (K League), 5 seasons; 40 transfers, all intra-league.
- Episode-level generation, not full matches.
- On-ball events only.
- Long rollouts are exposed to [[teacher-forcing|exposure bias]], which neither paper measures.
- Causal validity is not established — see [[counterfactual-simulation]].

## See Also

- [[eventgpt]]
- [[counterfactual-simulation]]
- [[large-event-model]]
- [[player-embedding]]
- [[tokenization]] · [[constrained-decoding]] · [[multi-task-learning]]
- [[gpt]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
