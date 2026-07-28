---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Control

A surface giving, for each location on the field, the probability that a given team would control the ball there. It turns 22 point positions into a continuous map of spatial dominance.

The vault holds **two independent constructions** of this object, used as inputs to different downstream value models and never compared against each other.

## Tradition 1: Gaussian Influence (Fernández & Bornn)

Two stages.

**Pitch influence** measures one player's reach. Each player is assigned a bivariate normal whose mean and covariance are adjusted for velocity and distance to the ball — so a sprinting player's influence stretches ahead of him. Normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i, t)} \in [0, 1]$$

**Pitch control** contests the two teams' influence, squashed to a probability:

$$PC(p, t) = \sigma\!\left(\gamma\Big(\lambda_1 \sum_i I_i(p,t) - \lambda_2 \sum_j I_j(p,t)\Big)\right)$$

with $\lambda_1 = \lambda_2 = \gamma = 1$ in the source — no team weighting, no shrinkage, nothing fitted.

## Tradition 2: Arrival-Time Contest (Spearman)

A physical model rather than a statistical one, and the control term inside [[obso|OBSO]].

Control is treated as a **Poisson point process**: the longer a player is near the ball uncontested, the more likely a controlled touch becomes. For player $j$ at location $r$:

$$\frac{d\,PPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big) f_j(t, r, T|s)\, \lambda_j$$

$f_j$ is a logistic function of $T$ minus expected intercept time, with temporal uncertainty $s = 0.45$ s; $\lambda_j = 4.3$ is the control rate. Both parameters come from Spearman et al. (2017) and are used unchanged by [[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]]

The leading bracket is the crucial term: **control probability is shared, so one player gaining it removes it from everyone else.** Integrating over $T$ and summing across a team gives that team's control at $r$.

## The Two Compared

| | Gaussian influence | Arrival-time (PPCF) |
|---|---|---|
| Mechanism | Influence as a density | Time-to-reach contest |
| Grounding | Statistical | **Physical** |
| Competition | Difference of summed influences | **Shared probability mass** |
| Parameters | Set to 1, unfitted | Fitted from data (2017) |
| Used by | [[expected-value-possession-framework\|Fernández et al.]] EPV | [[obso]], and hence [[c-obso]] |

Spearman's is more principled about *why* control occurs — arrival time is a real physical quantity, and the shared-mass term correctly makes control zero-sum. Fernández & Bornn's is cheaper, smoother, and additive.

The additivity is the substantive difference. Summing Gaussian influences **overstates control where coverage overlaps** — two defenders covering the same zone contribute independently, when in reality the second adds little. PPCF's shared bracket handles this by construction.

**No source in this vault compares them**, and both feed value models whose outputs are compared. A systematic difference propagates silently into [[obso|OBSO]], [[c-obso]], and the EPV surfaces alike. Given the additivity issue, the two are unlikely to agree in congested areas — precisely the areas that matter.

## What It Is For

Pitch control is **infrastructure** rather than an end in itself:

- In [[expected-value-possession-framework|the EPV framework]] it enters as an input feature and supplies the quantitative definition of *pressure* — an action is "under pressure" when the ball carrier's pitch control falls below 0.4. That definition is what makes the risk analysis possible: actions under pressure show a wider EPV-added distribution with more negative mass.
- In [[obso|OBSO]] it is one of three multiplied terms, answering "would the attacking team control the ball here?"

More broadly it is one of the few tools here that values **space rather than events**. Every event-data framework is blind to a player who creates a passing lane by dragging a marker away; a control surface sees it directly.

## Not the Same as Off-Ball Value

Frequently confused. Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for.

[[obso|OBSO]] makes the relationship explicit by multiplying them: control × transition × score.

## Assumptions Worth Knowing

- **Control is counterfactual everywhere except the ball's location.** The surface says who *would* win the ball there, and is validated only through downstream model performance, never directly.
- **Influence additivity** overstates control under overlapping coverage (Gaussian tradition only).
- **Unfitted parameters.** $\lambda_1$, $\lambda_2$, $\gamma$ are left at 1 in the Gaussian version, so nothing is tuned to make control predict anything.
- **Instantaneous.** Both treat the current snapshot; neither models where players are *going* beyond first-order velocity. [[trajectory-prediction]] is the natural complement and is not combined with either.

## See Also

- [[obso]] · [[probability-surface]] · [[off-ball-value]] · [[c-obso]]
- [[expected-possession-value]] · [[soccermap]] · [[trajectory-prediction]] · [[tactical-analysis]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]]
- [[optical-tracking-data]] · [[dynamic-pressure-lines]]
- [[expected-value-possession-framework|Soccer EPV Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
