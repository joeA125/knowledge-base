---
title: "Sig-Model"
type: concept
tags: [sports-analytics, path-signature, event-prediction, event-stream-data, feature-engineering, deep-learning, action-valuation]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Sig-Model

The Sig-Model ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]) predicts the next action type and its $(x,y)$ location in a football possession by encoding the possession as a [[path-signature]]. It outperforms the [[seq2event]] transformer benchmark on most metrics while running roughly 2.5× faster with a plain feedforward network.

## The Design Argument

Two departures from [[seq2event]] and [[nmstpp]], both following from one idea — that the **possession**, not a fixed window, is the natural unit.

### No fixed historical window
A fixed window of the last $k$ actions inevitably spans possession boundaries, mixing in either both teams' actions or one team's with spatio-temporal gaps where the opponent's possession was excised. It also throws away possession length, which carries tactical meaning.

Using whole possessions means variable-length, irregularly-sampled sequences — which is exactly what signatures handle natively and what recurrent and transformer encoders do not.

Supporting evidence: sweeping Seq2Event's window across 5, 10, and 40 actions produces **no clear optimum**, suggesting the window size is arbitrary rather than tuned.

### No handcrafted features
Inputs are only what is directly observed: the raw triplet $(x, y, T)$, the action sequence, and current score advantage. No shot angles, no distances to goal, no action durations — the signature encodes that geometry implicitly. See [[feature-engineering]].

## Architecture

Deliberately minimal:

| Stage | Detail |
|---|---|
| Continuous path | $(x, y, T)$ → time + visibility augmentation → linear interpolation → order-3 log-signature (**55 dims**) |
| Action sequence | Embedding → weighted average, weights $\propto 1/\text{position}$ (**7 dims**) |
| Context | Current score advantage, *scrad* (**1 dim**) |
| Network | Concatenate (63) → dense 256 → LeakyReLU(0.2) → dense 256 → LeakyReLU(0.2) → dense 9 |
| Output | 7 action logits + $x$ + $y$ |

Loss: $L(\theta) = \text{RMSE}_{(x,y)} + \lambda\,\text{CEL}_{\text{actions}}$ with $\lambda = 1$. Goals, possession changes, and match-end events carry zero CEL weight, being contextual rather than indicative of style.

The weighted average over action embeddings is a notably crude sequence encoder — it is the signature, not the network, doing the heavy lifting on temporal structure.

## Results

Against Seq2Event across five forecasting start points ($n_r \in \{3,\dots,7\}$), the pattern is consistent:

- **Sig-Model wins** on total loss, MSE (location), Brier score, and [[kl-divergence|KL divergence]].
- **Seq2Event wins narrowly** on CEL (action type), by roughly 0.001–0.002.

So the signature representation localises the next action distinctly better while classifying its type very slightly worse. Since the total loss is dominated by location error, Sig-Model wins overall.

Runtime 195–281s versus 250–688s, the gap widening when forecasting starts earlier and more predictions are made.

## Trained on a Laptop

The whole evaluation ran on a 2023 MacBook Pro (M3 Pro, 36GB RAM, 12 cores). Set against [[martingale-epv]]'s 461 processors, this is a useful reminder that model sophistication and computational demand are not the same axis — a well-chosen representation can substitute for scale.

## Limitations

- **Interevent time is not modelled**, unlike [[nmstpp]]. The authors say adding it is straightforward and leave it to future work — but it means Sig-Model cannot support timing-dependent metrics.
- The action-embedding weighted average uses fixed $1/\text{position}$ weights rather than learned ones.
- Event stream data only; the authors identify signatures over all 22 players' tracking trajectories as the natural extension.

## See Also

- [[path-signature]]
- [[lpv]]
- [[seq2event]]
- [[nmstpp]]
- [[feature-engineering]]
- [[kl-divergence]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
