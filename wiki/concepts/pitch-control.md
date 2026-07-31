---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis, theory-based-modelling]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 22%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Control

A surface giving, for each location on the field, the probability that a given team would control the ball there. It turns 22 point positions into a continuous map of spatial dominance.

The vault holds **two independent constructions**, used as inputs to different downstream value models and never compared against each other. See [[pitch-control-traditions-compared]] for where they should be expected to disagree and how to test it.

## Tradition 1: Arrival-Time Contest (Spearman)

The **potential pitch control field**, from [[beyond-expected-goals|Spearman (2018)]]. A physical model.

Control is a **Poisson point process** — a player near the ball uncontested becomes progressively more likely to make a controlled touch:

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T \mid s)\, \lambda_j$$

The leading bracket makes control **zero-sum**: probability mass gained by one player is removed from everyone else.

**Reachability.** $f_j$ is a **logistic** CDF over the residual between expected and true intercept time, with intercept time assuming constant acceleration 7 m/s² to a maximum 5 m/s. Logistic is chosen over normal *deliberately for its heavier tails* — absorbing tracking error, player facing, awareness and tactical decisions without modelling any of them.

**Ball flight time.** Trajectories are simulated with **aerodynamic drag** (Asai & Seo, 2013), and $PPCF_j$ is held at zero until the ball could physically arrive. The flight time selected is the one **most advantageous to the attackers**.

**Defensive advantage.** $\kappa = 1.72$ scales the control rate for defenders, since a defender is satisfied with heading clear while an attacker needs a controlled touch. Unique to this construction.

**Offside.** $\lambda_i$ is set to zero for attackers in offside positions.

Fitted: $s = 0.54$ s, $\lambda = 3.99$ Hz, $\kappa = 1.72$, by MAP with priors from the 2017 fit.

## Tradition 2: Gaussian Influence (Fernández & Bornn)

Statistical rather than physical. **Pitch influence** assigns each player a bivariate normal whose mean and covariance adjust for velocity and distance to the ball, normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

**Pitch control** contests the two teams' influence:

$$PC(p, t) = \sigma\!\left(\gamma\Big(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\Big)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ — no team weighting, no shrinkage, nothing fitted.

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

The substantive difference is **how they saturate**. Both do, by different mechanisms. F&B saturates through the sigmoid on a *difference* of summed influences, so adding a second defender to an already-dominated zone still moves the value. Spearman saturates on *total* control: once $\sum_k PPCF_k \to 1$, every remaining player's contribution is multiplied by approximately zero.

The consequence is that **F&B assigns more extreme values in crowded areas**, and the two should disagree in proportion to local player density — worst in the penalty area and around the ball.

A second difference is **ball travel time**: PPCF makes a location twenty metres away genuinely harder to reach than one five metres away, *for both teams*. The Gaussian construction treats arrival as instantaneous.

**No source in this vault compares them**, and both feed value models whose outputs *are* compared. See [[pitch-control-traditions-compared]] for the directional predictions and a proposed test — including the step that actually matters, which is whether substituting one surface for the other changes [[obso|OBSO]] player rankings.

## The Integration Horizon

An easily-missed parameter with real consequences. PPCF is defined as a differential equation in $T$ and must be integrated to some limit.

[[obso|Spearman]] integrates to $T \to \infty$: the question is who would *eventually* control the ball at that location.

[[xsot|Yeung & Fujii]] integrate only to **finite $T$ — the ball's travel time from the shooter to that attacker.** Their reasoning: even if an attacker gains control *after* the ball would have arrived, they cannot shoot from there. Control that arrives late is worthless.

This is a genuine improvement for any use of PPCF as a **passing-option** model rather than a general control surface. The infinite-horizon version systematically **over-values** distant or contested options, since it credits control the passer could never actually exploit. The same correction applies wherever a control surface is read as "could I play the ball here *now*" rather than "who owns this space".

Note this compounds with a data limitation in their case: StatsBomb 360 supplies positions per event and **no velocities**, which are set to zero — degrading the intercept-time estimate that PPCF depends on.

## What It Is For

Pitch control is **infrastructure** rather than an end in itself:

- In [[obso|OBSO]] it appears **twice** — as the control term, and inside the transition term raised to a power, since passers prefer destinations they can control.
- In [[xsot|xOSOT]] it discounts each teammate's shot-on-target probability by their chance of receiving the ball.
- In [[expected-value-possession-framework|the EPV framework]] it supplies the quantitative definition of *pressure* — an action is "under pressure" when the carrier's control falls below 0.4.

More broadly it is one of the few tools here that values **space rather than events**. Every event-data framework is blind to a player who opens a passing lane by dragging a marker away.

## Not the Same as Off-Ball Value

Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for. [[obso|OBSO]] makes the relationship explicit by multiplying them.

## Assumptions Worth Knowing

- **Control is counterfactual everywhere except the ball's location**, and is validated only through downstream model performance.
- **Influence additivity** over-states control under overlapping coverage (Gaussian tradition only).
- **Unfitted parameters** in the Gaussian version.
- **Instantaneous state.** Both use the current snapshot with first-order velocity; neither models where players are *going*. [[trajectory-prediction]] is the natural complement and is not combined with either.
- **Velocity is sometimes unavailable**, as in the StatsBomb 360 case above, in which case the reachability model degrades to positions alone.

## See Also

- [[pitch-control-traditions-compared]] — open question: do the two agree?
- [[obso]] · [[probability-surface]] · [[off-ball-value]] · [[c-obso]] · [[xsot]] · [[space-creation]]
- [[expected-possession-value]] · [[soccermap]] · [[trajectory-prediction]] · [[tactical-analysis]] · [[theory-based-modelling]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[calvin-yeung]]
- [[optical-tracking-data]] · [[dynamic-pressure-lines]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
