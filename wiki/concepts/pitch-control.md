---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis, theory-based-modelling]
sources: [raw/papers/physics_based_pass_probabilities.md, raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/team_defense_positioning_statsbomb.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 15%
  generated: 8%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Control

A surface giving, for each location on the field, the probability that a given team would control the ball there. It turns 22 point positions into a continuous map of spatial dominance.

The vault holds **two independent constructions**.

> ### `pitch-control-traditions-uncompared`
>
> **No source compares the two pitch-control traditions, and both feed value models whose outputs *are* compared.**
>
> ^[generated: an absence claim. rests-on: absence:no-held-source-compares-ppcf-and-gaussian — ⚠️ re-checked on ingest of Spearman et al. (2017), the origin of the first tradition: still holds. Expires on ingest of any paper computing both. Also on [[pitch-control-traditions-compared]] and the synthesis.]

## Tradition 1: Arrival-Time Contest (Spearman)

> **Correction, 2026-07-27.** This page previously treated PPCF as [[obso|OBSO's]] control term. It is older and more general. [[physics-based-pass-probabilities|Spearman et al. (2017)]] built a **pass-reception model**; the pitch control function is an *extension* of it — evaluate the stationary reception probability for an imaginary ball at every location. Control is a by-product of asking who receives passes, not a primitive.

Two physical components.

**Time to intercept.** Solve the player's equation of motion under $|\dot{r}| \le v_{max}$, $|\ddot{r}| \le a_{max}$ for the minimum time to reach each point on the ball's trajectory. Temporal uncertainty $\sigma$ absorbs differing speeds, reaction times and effort — deliberately unmodelled — through a **logistic** CDF, chosen for its heavy tails.

**Time to control.** A player near the ball controls it at rate $\lambda$, giving $P(t) = 1 - e^{-\lambda t}$. At $\lambda = 4.30$, ~95% chance of control within one second.

Combined recursively:

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T \mid s)\, \lambda_j$$

The leading bracket makes control **zero-sum**: mass gained by one player is removed from everyone else.

**Ball trajectory is simulated, not observed** — drag with constant $C_D = 0.25$, Magnus force ignored, so modelled balls travel straight. That choice is what allows *hypothetical* passes to be evaluated, and is the model's deepest design decision.

### The parameter chain, resolved

| Paper | $\sigma$ | $\lambda$ | Provenance |
|---|---|---|---|
| [[physics-based-pass-probabilities\|Spearman et al. (2017)]] | **0.45** | **4.30** | Fitted by MLE, with stat and syst errors |
| [[beyond-expected-goals\|Spearman (2018)]] | **0.54** | **3.99** | Refitted by MAP; priors 0.5 and 4.2 |
| [[c-obso]], [[drso]] | 0.45 | 4.30 | The 2017 values, citing 2018 |

The Fujii-group values are **legitimate published fits attributed to the wrong paper** — a bibliographic error, not a numerical one. Spearman (2018) adds $\kappa = 1.72$ as a defensive advantage; [[drso|DRSO]]'s goalkeeper multiplier ($\lambda = 12.9$) appears in neither Spearman paper and is a Fujii-group addition.

## Tradition 2: Gaussian Influence (Fernández & Bornn)

Statistical rather than physical. Each player gets a bivariate normal adjusted for velocity and distance to the ball, normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

$$PC(p, t) = \sigma\!\left(\gamma\Big(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\Big)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ — nothing fitted.

## The Two Compared

| | Arrival-time (PPCF) | Gaussian influence |
|---|---|---|
| Origin | A **pass-reception model** | A spatial-dominance model |
| Mechanism | Time-to-reach contest | Influence as a density |
| Grounding | **Physical** — seconds and hertz | Statistical |
| Saturates via | **Shared probability mass** | The sigmoid, on an influence *difference* |
| Ball travel time | **Modelled, with drag** | Not modelled |
| Attack/defence asymmetry | $\kappa = 1.72$ (2018 only) | None |
| Parameters | **Fitted, with stated errors** | Set to 1 |
| Validated against | **Actual pass receivers, 81%/68%** | Downstream model performance only |
| Used by | [[obso]], [[c-obso]], [[drso]], [[xsot\|xOSOT]] | [[expected-value-possession-framework\|Fernández et al.]] EPV |

The **validation row** is new and is the sharpest difference. PPCF was fitted and tested against **who actually received 5,471 held-out passes** — a direct check on a directly observable quantity. The Gaussian construction has no such validation at any point; its parameters are set to 1 and its correctness is inferred only from downstream EPV performance.

> ^[generated: the saturation analysis below is derived here from the two published formulations; the predicted direction of disagreement is untested. rests-on: source:ppcf-shared-bracket, source:fb-sigmoid-difference, claim:pitch-control-traditions-uncompared]

The substantive difference is **how they saturate**. F&B saturates through the sigmoid on a *difference* of summed influences, so a second defender in an already-dominated zone still moves the value — $\sigma(2) = 0.88$ against $\sigma(1) = 0.73$. Spearman saturates on *total* control: once $\sum_k PPCF_k \to 1$, every remaining contribution is multiplied by approximately zero.

If that reading is right, **F&B assigns more extreme values in crowded areas**, and the two should disagree in proportion to local player density — worst in the penalty area and around the ball. See [[pitch-control-traditions-compared]].

## The Integration Horizon

PPCF is a differential equation in $T$ and must be integrated to some limit.

The 2017 paper integrates to $T \to \infty$ — who would *eventually* receive the pass. [[obso|Spearman (2018)]] keeps that. [[xsot|Yeung & Fujii]] integrate only to **the ball's travel time**, on the reasoning that control gained after the ball would have arrived is useless for shooting.

A genuine improvement for any use of PPCF as a **passing-option** model: the infinite-horizon version systematically **over-values** distant or contested options.

## What It Is For

Infrastructure rather than an end in itself:

- In [[obso|OBSO]] it appears **twice** — as the control term, and inside the transition term raised to a power.
- In [[xsot|xOSOT]] it discounts each teammate's shot-on-target probability.
- In [[drso|DRSO]] it underpins EF-OBSO, computed on broadcast freeze frames.
- In [[expected-value-possession-framework|the EPV framework]] it defines *pressure* — control below 0.4.

The 2017 paper's own use is more direct: **corner-kick analysis**, where the defending team exerts 4% less control within 5 m of goal when a goal is scored than when the shot is saved.

## Not the Same as Off-Ball Value

Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. [[obso|OBSO]] makes the relationship explicit by multiplying them.

## Assumptions Worth Knowing

- **Magnus force ignored** in the trajectory model, so curved passes are mismodelled. Plausibly part of the 2017 model's 11-point completion-rate underestimate.
- **Rolling friction not modelled**; aerodynamic drag applies even after ground contact.
- **Influence additivity** over-states control under overlapping coverage (Gaussian tradition only).
- **Unfitted parameters** in the Gaussian version.
- **Instantaneous state.** Neither models where players are *going*; [[trajectory-prediction]] is the natural complement and is combined with neither.
- **Velocity is sometimes unavailable** — [[drso|DRSO]] assumes it, and tests five settings.

## See Also

- [[pitch-control-traditions-compared]] · [[tracking-error-propagation]] — open questions
- [[obso]] · [[probability-surface]] · [[off-ball-value]] · [[c-obso]] · [[drso]] · [[xsot]] · [[space-creation]]
- [[expected-possession-value]] · [[soccermap]] · [[trajectory-prediction]] · [[tactical-analysis]] · [[theory-based-modelling]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[optical-tracking-data]]
- [[physics-based-pass-probabilities|Spearman 2017 Summary]] · [[beyond-expected-goals|Spearman 2018 Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]]
