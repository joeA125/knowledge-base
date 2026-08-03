---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis, theory-based-modelling]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 15%
  generated: 10%
  imported: 0%
  ambiguous: 5%
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
> ^[generated: an absence claim. rests-on: absence:no-held-source-compares-ppcf-and-gaussian — ⚠️ expires on ingest of any paper computing both, or any survey of spatial models. Also on [[pitch-control-traditions-compared]] and the synthesis.]

## Tradition 1: Arrival-Time Contest (Spearman)

The **potential pitch control field**, from [[beyond-expected-goals|Spearman (2018)]]. A physical model.

Control is a **Poisson point process** — a player near the ball uncontested becomes progressively more likely to make a controlled touch:

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T \mid s)\, \lambda_j$$

The leading bracket makes control **zero-sum**: probability mass gained by one player is removed from everyone else.

**Reachability.** $f_j$ is a **logistic** CDF over the residual between expected and true intercept time, with intercept time assuming constant acceleration 7 m/s² to a maximum 5 m/s. Logistic is chosen over normal *deliberately for its heavier tails* — absorbing tracking error, player facing, awareness and tactical decisions without modelling any of them.

**Ball flight time.** Trajectories are simulated with **aerodynamic drag** (Asai & Seo, 2013), and $PPCF_j$ is held at zero until the ball could physically arrive. The flight time selected is the one **most advantageous to the attackers**.

**Defensive advantage.** $\kappa = 1.72$ scales the control rate for defenders, since a defender is satisfied with heading clear while an attacker needs a controlled touch.

**Offside.** $\lambda_i$ is set to zero for attackers in offside positions.

Fitted: $s = 0.54$ s, $\lambda = 3.99$ Hz, $\kappa = 1.72$, by MAP with priors from the 2017 fit.

## Tradition 2: Gaussian Influence (Fernández & Bornn)

Statistical rather than physical. **Pitch influence** assigns each player a bivariate normal whose mean and covariance adjust for velocity and distance to the ball, normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

**Pitch control** contests the two teams' influence:

$$PC(p, t) = \sigma\!\left(\gamma\Big(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\Big)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ — nothing fitted.

## The Two Compared

| | Arrival-time (PPCF) | Gaussian influence |
|---|---|---|
| Mechanism | Time-to-reach contest | Influence as a density |
| Grounding | **Physical** — seconds and hertz | Statistical |
| Saturates via | **Shared probability mass** | The sigmoid, on an influence *difference* |
| Ball travel time | **Modelled, with drag** | Not modelled |
| Attack/defence asymmetry | **$\kappa = 1.72$** | None |
| Offside | **Handled** | Not handled |
| Parameters | Fitted from measured priors | Set to 1 |
| Used by | [[obso]], [[c-obso]], [[xsot\|xOSOT]] | [[expected-value-possession-framework\|Fernández et al.]] EPV |

> ^[generated: the saturation analysis below is derived here from the two published formulations, and the predicted direction of disagreement is untested. rests-on: source:ppcf-shared-bracket, source:fb-sigmoid-difference, claim:pitch-control-traditions-uncompared — cascades if a comparison is found, since the analysis would then be superseded by measurement]

The substantive difference is **how they saturate**. Both do, by different mechanisms. F&B saturates through the sigmoid on a *difference* of summed influences, so adding a second defender to an already-dominated zone still moves the value — $\sigma(2) = 0.88$ against $\sigma(1) = 0.73$. Spearman saturates on *total* control: once $\sum_k PPCF_k \to 1$, every remaining player's contribution is multiplied by approximately zero.

If that reading is right, **F&B assigns more extreme values in crowded areas**, and the two should disagree in proportion to local player density — worst in the penalty area and around the ball.

A second difference is **ball travel time**: PPCF makes a location twenty metres away genuinely harder to reach than one five metres away, *for both teams*. The Gaussian construction treats arrival as instantaneous.

See [[pitch-control-traditions-compared]] for the directional predictions and the decisive step — whether substituting one surface for the other changes [[obso|OBSO]] player rankings.

## The Integration Horizon

PPCF is a differential equation in $T$ and must be integrated to some limit.

[[obso|Spearman]] integrates to $T \to \infty$: who would *eventually* control the ball there.

[[xsot|Yeung & Fujii]] integrate only to **finite $T$ — the ball's travel time.** Even if an attacker gains control *after* the ball would have arrived, they cannot shoot from there.

This is a genuine improvement for any use of PPCF as a **passing-option** model rather than a general control surface: the infinite-horizon version systematically **over-values** distant or contested options.

Note this compounds with a data limitation in their case: StatsBomb 360 supplies no velocities, which are set to zero — degrading the intercept-time estimate PPCF depends on. See [[tracking-error-propagation]].

## What It Is For

Pitch control is **infrastructure** rather than an end in itself:

- In [[obso|OBSO]] it appears **twice** — as the control term, and inside the transition term raised to a power, since passers prefer destinations they can control.
- In [[xsot|xOSOT]] it discounts each teammate's shot-on-target probability by their chance of receiving the ball.
- In [[expected-value-possession-framework|the EPV framework]] it supplies the quantitative definition of *pressure* — control below 0.4.

More broadly it is one of the few tools here that values **space rather than events**.

## Not the Same as Off-Ball Value

Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for. [[obso|OBSO]] makes the relationship explicit by multiplying them.

## Assumptions Worth Knowing

- **Control is counterfactual everywhere except the ball's location**, and is validated only through downstream model performance.
- **Influence additivity** over-states control under overlapping coverage (Gaussian tradition only).
- **Unfitted parameters** in the Gaussian version.
- **Instantaneous state.** Neither models where players are *going*; [[trajectory-prediction]] is the natural complement and is combined with neither.
- **Velocity is sometimes unavailable**, in which case reachability degrades to positions alone.

## See Also

- [[pitch-control-traditions-compared]] · [[tracking-error-propagation]] — open questions
- [[obso]] · [[probability-surface]] · [[off-ball-value]] · [[c-obso]] · [[xsot]] · [[space-creation]]
- [[expected-possession-value]] · [[soccermap]] · [[trajectory-prediction]] · [[tactical-analysis]] · [[theory-based-modelling]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[calvin-yeung]] · [[optical-tracking-data]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
