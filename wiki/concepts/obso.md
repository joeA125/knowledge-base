---
title: "OBSO (Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, sports-analytics, pitch-control, probability-surface, optical-tracking-data, player-evaluation, action-valuation, spatiotemporal]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# OBSO (Off-Ball Scoring Opportunity)

[[william-spearman|Spearman's]] (2018) metric for valuing a player who does not have the ball, by asking: **if the next on-ball event happened at this location, how likely is a goal?**

The vault has cited OBSO as a dependency of two separate lines — the [[keisuke-fujii|Fujii group's]] off-ball and defensive work, and Spearman's own — without holding a description of it. [[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] supply the first primary account here, including the equations.

## The Factorisation

OBSO is a sum over pitch locations of a joint probability, which under an independence assumption factorises into three interpretable terms:

$$P(G|D) = \sum_{r \in R \times R} P(S_r|D)\, P(C_r|D)\, P(T_r|D)$$

where $D$ is the instantaneous game state — all player positions and velocities.

| Term | Question | Model |
|---|---|---|
| $P(T_r\|D)$ | Will the next event happen *here*? | 2-D Gaussian centred on the ball, σ = 14 m (the average distance to the next event) |
| $P(C_r\|D)$ | Would the attacking team *control* it here? | **PPCF** — potential pitch control field |
| $P(S_r\|D)$ | Would a goal follow from here? | Decreasing function of distance to goal |

The structure is worth noting: it is a [[structured-model-decomposition|decomposition]] in the same spirit as [[expected-value-possession-framework|Fernández et al.'s]], reached independently and earlier, and far simpler — three rule-based or lightly-fitted terms rather than nine trained networks.

## PPCF: Control as a Poisson Process

The control term is the substantive piece, and it is a physical model rather than a learned one.

The idea: a player's ability to make a controlled touch is a **Poisson point process** — the longer they are near the ball uncontested, the more likely control becomes. For player $j$ at location $r$:

$$\frac{d\,PPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big) f_j(t, r, T|s)\, \lambda_j$$

$f_j$ is the probability player $j$ can *reach* $r$ within time $T$ — a logistic function of $T$ minus expected intercept time, with temporal uncertainty $s = 0.45$ s. $\lambda_j = 4.3$ is the control rate. The leading bracket is the crucial coupling: **control probability is shared, so one player gaining it removes it from everyone else.**

Integrating over $T$ and summing across the attacking team gives $P(C_r|D)$.

This is the same conceptual object as [[pitch-control|Fernández & Bornn's pitch control]], arrived at differently. Spearman models an **arrival-time contest** from physics; Fernández & Bornn model **influence as a Gaussian density**. Spearman's is more principled about the mechanism; Fernández & Bornn's is cheaper and smoother. The vault holds both, and no source compares them.

## The Weak Term, and Its Repair

$P(S_r|D)$ — distance to goal alone — is by far the crudest of the three. It ignores angle, defenders, and the goalkeeper.

[[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] replace it with a **potential score model**: shot value computed per-degree across the shooting angle, discounted by a shot-blocking distribution built from Gaussians on each goal-side defender (variance widening with distance, goalkeepers weighted double).

Validated on 494 shots: RMSE **0.309** against **0.324** ($p < 10^{-10}$). A modest gain, but the qualitative difference is sharper — two shots from equal distance score identically under the original model and 0.0489 vs 0.1202 under the replacement, according to defender congestion.

Note what this makes OBSO's score term into: essentially a geometric [[expected-goals|xG]] model. The vault now holds three xG formulations — logistic on shot features, tracking-augmented with blockage counts, and this angular-integration one — none benchmarked against the others.

## What OBSO Values, and What It Does Not

**Values:** the player who would *receive* the ball, at the location they occupy. A striker drifting into a dangerous pocket scores highly.

**Does not value:** the player whose movement *created* that pocket for someone else. OBSO is egocentric — it asks what your position is worth to you.

This is precisely the gap [[c-obso]] fills, and it is why C-OBSO's correlation with salary (ρ = 0.45) while plain OBSO's is negative and non-significant (ρ = −0.28) is the headline result of that paper.

## Position in the Vault

| | [[obso\|OBSO]] | [[pitch-control]] | [[probability-surface\|Pass EPV surface]] |
|---|---|---|---|
| Question | What is a goal worth from here, if the ball arrives? | Who would win the ball here? | What is the possession worth if passed here? |
| Estimation | Rule-based / physical | Physical or Gaussian | Learned ([[soccermap]]) |
| Whose value | The receiver's | Neither team's, per se | The possessing team's |
| Cost | Low | Low | High |

All three are surfaces over the pitch, and all three are read at player positions to produce off-ball value. They differ in what the surface *means*.

OBSO is the substrate for both Fujii-group counterfactual lines — [[c-obso]] for attacking space creation, and the Umemoto & Fujii (2023) defensive positioning work. That makes Spearman (2018) itself a notable remaining gap in `raw/`.

## See Also

- [[c-obso]] · [[pitch-control]] · [[off-ball-value]] · [[probability-surface]]
- [[expected-goals]] · [[structured-model-decomposition]] · [[optical-tracking-data]]
- [[william-spearman]] · [[keisuke-fujii]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
