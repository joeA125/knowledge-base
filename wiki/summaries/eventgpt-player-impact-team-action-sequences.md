---
title: "EventGPT: Capturing Player Impact from Team Action Sequences Using GPT-Based Framework — Source Summary"
type: source_summary
tags: [sports-analytics, counterfactual, transformer, event-prediction, player-evaluation, action-valuation, entity-embedding, teacher-forcing, event-stream-data]
sources: [raw/papers/eventgpt-player-impact-from-team-action-sequences.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# EventGPT: Capturing Player Impact from Team Action Sequences

**Authors:** [[minho-lee]]\*, [[miru-hong]]\*, [[geonhee-jo]], [[jae-hee-so]], [[pascal-bauer]], [[sang-ki-ko]] (\*equal contribution)
**Affiliations:** Saarland University; University of Seoul; Bank of Korea; Deutscher Fussball-Bund
**Published:** 2025

## Key Contribution

[[eventgpt]] — a player-conditioned, value-aware next-event prediction model on a nanoGPT decoder-only transformer. Its distinguishing contribution is the prediction target: **[[on-ball-value|residual On-Ball Value]] (rOBV)**, the expected *future* value a player contributes from the current moment to the end of the episode, rather than the immediate value of the action just taken.

This is the direct predecessor to [[scoutgpt]] by the same group.

## Why a Forward-Looking Value Target

The framing critique applies to the whole [[action-valuation]] literature:

> Value estimation is typically applied as a post-hoc layer on completed event sequences; the value of an action is evaluated after it happens, rather than co-learned with the sequential process that generates actions.

[[vaep]], [[expected-threat|xT]] and OBV all score actions *after the fact*. rOBV instead makes value a **prediction target inside the generative process**:

$$rOBV_t = \mathbb{E}\left[\sum_{\tau=t}^{T_{\text{episode}}} OBV_\tau \;\Big|\; \text{state at } t, \text{player } p_t\right]$$

Because it accumulates to the end of the episode, it captures how a player's presence shapes the *subsequent* structure of the possession — not just the action they are about to perform. That is what a transfer question actually asks.

## Method

### Tokenization and episodes
Uses [[spadl]] representation — the first model in this vault's football cluster to do so, where [[nmstpp]] and [[scoutgpt]] use WyScout and VERSA respectively. Each event becomes 7 tokens:

$$v_t = (h_t, e_t, x_t, y_t, \delta_t, o_t, rOBV_t)$$

Notably **rOBV is itself a token** in the sequence, discretised into the shared vocabulary alongside coordinates and time. Value is not a separate head but part of what gets generated — a cleaner integration than [[scoutgpt]]'s auxiliary heads, though the successor's approach performed better.

An **episode** is defined as a contiguous stretch where the 22 on-pitch players stay unchanged, breaking at set-pieces, goals, period transitions, substitutions and dismissals. This guarantees a consistent personnel context, which player-conditioned prediction requires.

A context block encodes all 22 player IDs, minute, cumulative goals, and cumulative cards.

### The critical design choice
> Importantly, the player identity $p_t$ conditions the prediction but is **never itself predicted**; this enables controlled counterfactual substitution by simply replacing the identity token while holding the surrounding context fixed.

This is the same mechanism [[scoutgpt]] uses, established here first. Without it the model could regenerate the player it expected, silently undoing the intervention.

### Architecture
nanoGPT decoder-only transformer with **weight tying** between the input embedding matrix and output projection. All attributes — categorical, spatial, temporal, and value — share one vocabulary and one embedding layer, so the transformer learns their interactions through attention rather than hand-designed feature fusion.

Trained with [[teacher-forcing|teacher forcing]] on next-token cross-entropy.

## Data

Premier League, 2020/21–2024/25: 1,900 matches, 173,951 episodes, 1,221 players, 22.48 events per episode.

*(The paper states the season range inconsistently — 2019/20–2023/24 in §3.1 against 2020/21–2024/25 in the abstract and Table 1. The five-season span is unambiguous; the exact window is not.)*

## Results

| Model | Team ↑ | Event type ↑ | $x$ ↓ | $y$ ↓ | Time ↓ | Success ↑ | rOBV ↓ |
|---|---|---|---|---|---|---|---|
| LEM | 85.20% | 74.07% | 9.01 | 8.11 | 1.37 | 90.51% | 0.014 |
| LEM Transformer | **96.05%** | 80.42% | 7.15 | 7.08 | 1.40 | 86.92% | **0.008** |
| **EventGPT** | 94.12% | **82.91%** | **4.30** | **4.31** | **1.11** | **92.87%** | 0.009 |

Spatial error roughly halves against the [[nmstpp]]-derived LEM Transformer baseline. The authors attribute the event-type gain to player conditioning: knowing *who* has the ball genuinely improves prediction of *what* they will do.

Worth noting they do **not** win on rOBV error — the baseline edges it (0.008 vs 0.009).

## Player Embeddings

Position labels are never used in training, yet t-SNE projection of the learned embeddings produces role-consistent clusters with sensible geometry: wing-backs sit between defenders and wingers; defensive midfielders bridge defence and attack; forwards, wingers and attacking midfielders form sub-clusters in the final third.

The embeddings support **similarity-based retrieval** for scouting, which Case Study 3 uses to generate candidate replacements automatically rather than from a hand-picked list. See [[player-embedding]].

## Counterfactual Case Studies

The four case studies are the paper's real substance, and they make a sharper argument than the headline metrics.

### 1. Striker fit is system-dependent
Substituting strikers into Haaland's Manchester City context and Højlund's Manchester United context **reverses rankings**. Haaland's predicted rOBV drops from 2.71 in his own context to 1.37 in Højlund's. Isak scores highest in both (3.76 and 4.65), which the authors read as adaptability — generating value without relying on heavy service.

The cross-substitution matrix makes the point systematically: Haaland is productive in City's structured build-up and declines sharply in transition-heavy environments.

### 2 & 3. Role-based scouting
Evaluating all right-wingers with >1,500 minutes in Saka's context puts Salah top (19.78 predicted rOBV against Saka's simulated 18.59), with a *more pass-inclusive* profile than Saka's dribble-led one — a different but viable reading of the role.

Case Study 3 retrieves candidates by embedding cosine similarity rather than manual selection, and finds instructive negatives: Mbeumo and Bowen are stylistically close but project much lower rOBV (11.80, 10.10). **Superficial similarity does not imply similar contribution in a specific context.**

### 4. High value does not transfer across roles
Substituting Haaland into four Arsenal defensive contexts collapses his predicted rOBV (1.37–2.35) well below the original defenders (3.63–8.78).

The methodological point is sharper than the football one: **the model has no positional labels**, so it cannot be penalising Haaland for "playing out of position". The decline comes purely from the contextual demands of those match situations.

## An Honest Aggregation Detail

Attackers have high-variance OBV distributions — many low-value actions, rare high-value ones. An arithmetic mean over Monte Carlo samples would be dominated by the frequent low-value outcomes and understate impact.

The authors therefore use a **truncated mean over the top quartile** for attackers, and arithmetic mean for other roles. This is defensible but is a hand-chosen, position-dependent estimator, and the two positions are not directly comparable as a result.

## Limitations

- **Simulation baselines are self-inconsistent.** Saka's simulated rOBV (18.59) exceeds his ground truth (15.72), and the authors use the *simulated* value as the comparison baseline — a positive bias they acknowledge but do not correct.
- **Training-window sensitivity.** Núñez's ground-truth rOBV was unusually low in 2023/24, and the authors note this propagates into the baseline against which substitutions are compared.
- On-ball events only; tracking data flagged as future work.
- Episode-level generation, not full matches.

## Relation to ScoutGPT

| | EventGPT (2025) | [[scoutgpt]] (2026) |
|---|---|---|
| Event format | [[spadl]] | VERSA (with validity correction) |
| Tokens per event | 7 | 10 |
| Value handling | rOBV **as a token** | Auxiliary **heads** at outcome positions |
| Decoding | Unconstrained | [[constrained-decoding\|Validity-masked]] |
| Counterfactual | Re-score fixed sequence under new player | **Re-generate** full sequence |
| League | Premier League | K League |

The key advance in ScoutGPT is generating the *whole* sequence under the new player, rather than re-scoring the observed one. EventGPT's substitution holds the event sequence fixed and only re-evaluates rOBV — so it captures how a player would value the same situations, not how they would change what happens.

## See Also

- [[eventgpt]]
- [[on-ball-value]]
- [[player-embedding]]
- [[teacher-forcing]]
- [[counterfactual-simulation]]
- [[scoutgpt]]
