---
title: "Pitch Value Model"
type: concept
tags: [probability-surface, sports-analytics, off-ball, deep-learning, spatiotemporal, optical-tracking-data, weak-supervision, tactical-analysis]
sources: [raw/papers/wide_open_spaces_creation_football.md]
confidence: 0.85
provenance:
  extracted: 75%
  inferred: 20%
  generated: 4%
  imported: 1%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Value Model

Learning **what each location on the pitch is worth**, conditioned on where the ball is.

[[wide-open-spaces-space-creation|Fernández & Bornn (2018)]] introduce it as the value half of their space metrics: control of a location is worth nothing without knowing what the location is worth. See [[space-occupation-gain]].

## The Inversion

The training signal is the novel part. There is no outcome label for "how valuable is this square metre" — so the model learns from **where defenders choose to stand**:

> *"considering a sufficiently high number of situations, the defending team distributes itself throughout the field in a manner which covers high value spaces."*

The target at each grid cell is the summed influence of all defending players, clipped at 1:

$$D_{k,l}(t) = \sum_d I_d(p_b(t), p_{k,l}(t)), \qquad \hat V_{k,l}(t) = \min(D_{k,l}(t), 1)$$

A feed-forward network with one hidden layer then learns $V_{k,l}(t) = f_n(p_b(t), p_{k,l}(t); \theta)$ — a mapping from ball position and cell position to value. Trained on 2.4M examples from 20 matches, MSE loss, sigmoid activation, Adam, 10-fold cross-validation.

**Defensive behaviour is treated as a revealed preference.** Rather than asking what is valuable, the model asks what professionals *act as though* is valuable, and takes their collective positioning as the answer. That needs no outcome labels at all — a form of [[weak-supervision]] where the supervision is another agent's judgement.

The same move appears elsewhere in the vault under a different name: [[policy-modelling]] estimates the policy players actually follow rather than an optimal one, and [[imitation-learning|imitation]] treats observed behaviour as the target. This is the spatial version — **infer the value function from the defence's revealed policy.**

## The Admitted Failure, and the Patch

The learned surface does not know that space is worth more further up the pitch. High-value locations in one's own first quarter score the same as high-value locations in the final quarter.

The fix is post-hoc: **multiply by a distance-to-goal normalisation surface.** The authors state the intuition it encodes — *"when one's own goalkeeper has the ball, the overall value of space is limited, but when in the opponent's box, space is much more valuable alongside the looming threat of a shot on goal."*

Worth being clear about what this means. The hypothesis — that defensive positioning reveals value — is **partly false**, and in a predictable direction. Defenders crowd their own box heavily, so the model learns that one's own penalty area is extremely valuable. It is, defensively; it is not valuable *to attack*. The normalisation corrects a systematic bias the training signal introduces rather than adding a refinement.

That is a real limitation of revealed-preference targets generally: **the behaviour reveals the agent's objective, not yours.** Learning attacking value from defensive positioning inherits the defence's priorities.

## Relation to the Vault's Other Value Surfaces

| | **Pitch value** | [[obso\|OBSO]] score term | [[probability-surface\|SoccerMap EPV]] |
|---|---|---|---|
| Question | What is this location worth? | Would a goal follow from here? | What is a pass here worth? |
| Trained on | **Defensive positioning** | Distance-to-goal curve | Observed pass outcomes |
| Needs outcome labels | **No** | Yes (goals) | Yes |
| Conditioned on ball | **Yes** | No | Yes |
| Corrected post-hoc | **Yes — distance to goal** | $\beta$ exponent | No |

The middle column is the interesting comparison. [[obso|Spearman's]] score term is *also* crude and *also* patched with a fitted exponent — and both authors say so. Two independent value models, both admitting their spatial value function is the weak component. See [[shot-value-formulations-compared]].

## Limitations

- **The revealed-preference hypothesis is partly false**, requiring the distance-to-goal patch — as above.
- **No outcome validation.** The model is never checked against whether possession at high-value locations actually produces goals; it is validated by expert inspection of the surfaces.
- **Ball position only.** Value is conditioned on the ball and nothing else — not on scoreline, phase, or where the other 21 players are, beyond what the defence taught it in training.
- **21 × 15 grid** for training targets, coarser than the surfaces it produces.
- **Single hidden layer**, chosen after linear models underperformed; no architecture search reported.

## See Also

- [[space-occupation-gain]] · [[pitch-control]] · [[probability-surface]] · [[space-creation]] · [[off-ball-value]]
- [[policy-modelling]] · [[imitation-learning]] · [[weak-supervision]] · [[obso]] · [[expected-value-possession-framework]]
- [[javier-fernandez]] · [[luke-bornn]] · [[shot-value-formulations-compared]]
- [[wide-open-spaces-space-creation|Source Summary]]
