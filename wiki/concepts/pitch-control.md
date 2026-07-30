---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
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

The vault holds **two independent constructions**, used as inputs to different downstream value models and never compared against each other.

## Tradition 1: Arrival-Time Contest (Spearman)

The **potential pitch control field**, from [[beyond-expected-goals|Spearman (2018)]], extending Spearman et al. (2017). A physical model.

Control is a **Poisson point process** — a player near the ball uncontested becomes progressively more likely to make a controlled touch:

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T \mid s)\, \lambda_j$$

The leading bracket is the crucial term: **control probability is shared, so one player gaining it removes it from everyone else.** Integrating over $T$ and summing across a team gives that team's control at $r$.

**Reachability.** $f_j$ is a **logistic** CDF over the residual between expected and true intercept time. Expected intercept time assumes constant acceleration 7 m/s² to a maximum 5 m/s. The logistic is chosen over the normal *deliberately for its heavier tails* — absorbing tracking error, player facing, awareness and tactical decisions without modelling any of them explicitly.

**Ball flight time.** Passes are not instantaneous. Trajectories are simulated with **aerodynamic drag** (Asai & Seo, 2013), and $PPCF_j$ is held at zero until the ball could physically arrive — so longer passes give both teams more time to converge. The flight time selected is the one **most advantageous to the attackers**, an explicit thumb on the scale compensating for weaknesses in the transition model.

**Defensive advantage.** A parameter $\kappa$ scales the control rate for defenders, since a defender is satisfied with heading clear while an attacker needs a controlled touch. Fitted at $\kappa = 1.72$ — a substantial asymmetry, and unique to this construction.

**Offside.** $\lambda_i$ is set to zero for attackers in offside positions.

Fitted parameters: $s = 0.54$ s, $\lambda = 3.99$ Hz, $\kappa = 1.72$, by MAP with priors drawn from the 2017 fit.

## Tradition 2: Gaussian Influence (Fernández & Bornn)

Two stages, statistical rather than physical.

**Pitch influence** measures one player's reach: a bivariate normal whose mean and covariance are adjusted for velocity and distance to the ball, so a sprinting player's influence stretches ahead of him. Normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

**Pitch control** contests the two teams' influence, squashed to a probability:

$$PC(p, t) = \sigma\!\left(\gamma\Big(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\Big)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ in the source — no team weighting, no shrinkage, nothing fitted.

## The Two Compared

| | Arrival-time (PPCF) | Gaussian influence |
|---|---|---|
| Mechanism | Time-to-reach contest | Influence as a density |
| Grounding | **Physical** — units of seconds and hertz | Statistical |
| Competition | **Shared probability mass** | Difference of summed influences |
| Ball travel time | **Modelled, with drag** | Not modelled |
| Attack/defence asymmetry | **$\kappa = 1.72$** | None |
| Offside | **Handled** | Not handled |
| Parameters | Fitted from measured priors | Set to 1 |
| Used by | [[obso]], and hence [[c-obso]] | [[expected-value-possession-framework\|Fernández et al.]] EPV |

The substantive difference is **additivity**. Summing Gaussian influences over-states control where coverage overlaps — two defenders covering one zone contribute independently, when in reality the second adds little. PPCF's shared bracket makes control zero-sum by construction.

A second difference is now visible from the primary source: PPCF models **ball travel time**, so a location twenty metres away is genuinely harder to reach than one five metres away *for both teams*. The Gaussian construction treats arrival as instantaneous.

**No source in this vault compares them**, and both feed value models whose outputs *are* compared. Given the additivity and travel-time differences, the two are least likely to agree in congested areas near goal — precisely the areas that matter. A systematic difference propagates silently into [[obso|OBSO]], [[c-obso]] and the EPV surfaces alike.

## What It Is For

Pitch control is **infrastructure** rather than an end in itself:

- In [[obso|OBSO]] it appears **twice** — as the control term, and inside the transition term raised to a power, since passers prefer destinations they can control. That coupling is easy to miss.
- In [[expected-value-possession-framework|the EPV framework]] it is an input feature and supplies the quantitative definition of *pressure* — an action is "under pressure" when the carrier's pitch control falls below 0.4. Actions under pressure show a wider EPV-added distribution with more negative mass.

More broadly it is one of the few tools here that values **space rather than events**. Every event-data framework is blind to a player who opens a passing lane by dragging a marker away; a control surface sees it directly.

## Not the Same as Off-Ball Value

Frequently confused. Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for.

[[obso|OBSO]] makes the relationship explicit by multiplying them: transition × control × score.

## Assumptions Worth Knowing

- **Control is counterfactual everywhere except the ball's location.** The surface says who *would* win the ball there, and is validated only through downstream model performance, never directly.
- **Influence additivity** over-states control under overlapping coverage (Gaussian tradition only).
- **Unfitted parameters** in the Gaussian version; nothing is tuned to make control predict anything.
- **Instantaneous state.** Both use the current snapshot with first-order velocity; neither models where players are *going*. [[trajectory-prediction]] is the natural complement and is not combined with either.

## See Also

- [[obso]] · [[probability-surface]] · [[off-ball-value]] · [[c-obso]] · [[space-creation]]
- [[expected-possession-value]] · [[soccermap]] · [[trajectory-prediction]] · [[tactical-analysis]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]]
- [[optical-tracking-data]] · [[dynamic-pressure-lines]]
- [[beyond-expected-goals|Spearman Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]]
