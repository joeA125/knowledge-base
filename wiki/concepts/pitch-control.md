---
title: "Pitch Control"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, off-ball, probability-surface, statistics, tactical-analysis, theory-based-modelling]
sources: [raw/papers/wide_open_spaces_creation_football.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/team_defense_positioning_statsbomb.md]
confidence: 0.9
provenance:
  extracted: 82%
  inferred: 12%
  generated: 4%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pitch Control

A surface giving, for each location on the field, the probability that a given team would control the ball there. It turns 22 point positions into a continuous map of spatial dominance.

The vault holds **two independent constructions, and now both origin papers.**

## What Both Replaced

Until the late 2010s the default was [[voronoi-tessellation|Voronoi tessellation]] — partition the pitch, assign each cell to its nearest player. Both traditions reject it for the same reason: **ownership is continuous, not discrete**, and the uncertain space *between* players is exactly where contests are decided.

They also reject it for a second reason and solve that differently: a proximity partition cannot express **distance to the ball**, which governs both how far a player's influence extends and whether a location matters at all.

> ### `pitch-control-traditions-uncompared`
>
> **No source compares the two traditions, and both feed value models whose outputs *are* compared.**
>
> ^[generated: an absence claim. rests-on: absence:no-held-source-compares-ppcf-and-gaussian — ⚠️ re-checked on ingest of both origin papers, 2026-07-27: still holds, and now structurally explained. Neither origin cites the other; both cite the Voronoi line. They are siblings framed against a common ancestor, not rivals. See [[voronoi-tessellation]].]

## Tradition 1: Arrival-Time Contest (Spearman)

Originating in [[physics-based-pass-probabilities|Spearman et al. (2017)]] as a **pass-reception model**; the control surface is that model evaluated for an imaginary stationary ball at every location. Control is a by-product of asking who receives passes. See [[pass-probability-model]].

Control is a **Poisson point process** — a player near the ball uncontested becomes progressively more likely to make a controlled touch:

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T \mid s)\, \lambda_j$$

The leading bracket makes control **zero-sum**: mass gained by one player is removed from everyone else.

**Reachability** is a **logistic** CDF over the residual between expected and true intercept time, chosen for heavy tails, absorbing differing speeds, reaction times and effort. **Ball flight** is simulated with drag; $PPCF_j$ stays at zero until the ball could physically arrive.

Fitted by MLE: $\sigma = 0.45$, $\lambda = 4.30$, both with stated statistical *and* systematic errors. [[obso|Spearman (2018)]] refits to 0.54 and 3.99 and adds $\kappa = 1.72$ for defenders and an offside rule.

## Tradition 2: Gaussian Influence (Fernández & Bornn)

Originating in [[wide-open-spaces-space-creation|Fernández & Bornn (2018)]] as a **spatial-dominance model**, built to support space-creation metrics.

**Player influence** is a bivariate normal normalised against its own peak:

$$I_i(p, t) = \frac{f_i(p, t)}{f_i(p_i(t), t)}$$

The covariance is constructed by rotation and scaling — rotate to the velocity angle, stretch along it by $s^2/13^2$, and set the base radius from **distance to the ball: 4 m on the ball, rising to 10 m at 20 m away.** The mean translates forward by half the velocity vector.

That radius rule inverts naive intuition deliberately: a player *far* from the ball has *wider* influence, because if the ball travels toward him he has more time to cover more ground.

**Team control** is the logistic of the difference of summed influences:

$$PC(p, t) = \sigma\Big(\sum_i I_i(p,t) - \sum_j I_j(p,t)\Big)$$

### Saturation is design intent, not artefact

> **Superseded, 2026-07-27.** The vault previously marked the saturation analysis below as `generated` — a reading derived here from the published formulation. The origin paper states it outright, so it is now **extracted**.

> *"a single player without any influence of any other player at its current location only controls $\text{logistic}(1) = 0.73$ of the space. This provides the need of higher density of players near a given area to provide higher level of control in that area."*

So the sigmoid-on-a-difference is doing deliberate work: **a lone player does not own his own location**, and density is required for high control. That is the mechanism the vault predicted would make the two traditions diverge in crowded areas, confirmed as intentional.

### Parameters are expert-set, not unfitted

> **Superseded, 2026-07-27.** The vault recorded this model as having "parameters set to 1, unfitted." Too crude.

- $\gamma$ is an **acknowledged simplification** — *"we can include a constant within $\sigma$ to add more flexibility, if desired"*, and Equation 2 is described as *"a simplified version"*.
- The influence-radius function (4→10 m) **is** parameterised, *"based on the opinion of expert soccer analysts"*.
- Maximum speed 13 m/s; jogging threshold 1.5 m/s.

**Expert-set is a third position** between Spearman's MLE fits and genuinely unfitted. It matters for [[model-selection]]: expert-set parameters cannot inherit priors from prior measurement, but they are not arbitrary either — they encode domain judgement that is inspectable and disputable.

## The Two Compared

| | Arrival-time (PPCF) | Gaussian influence |
|---|---|---|
| Origin | A **pass-reception** model | A **spatial-dominance** model |
| Question answered | Who receives a pass here? | Who dominates this space? |
| Mechanism | Time-to-reach contest | Influence as a density |
| Continuity comes from | **Temporal uncertainty** on arrival | **Smoothness** of the influence field |
| Saturates via | Shared probability mass | The sigmoid, on a *difference* |
| Ball travel time | **Modelled, with drag** | Not modelled |
| Distance to ball | Via flight time | **Via influence radius, 4→10 m** |
| Attack/defence asymmetry | $\kappa = 1.72$ (2018 only) | None |
| Parameters | **MLE-fitted, with stated errors** | **Expert-set** |
| Validated against | **5,471 held-out pass receivers** (81%/68%) | **Expert video review**, no ground truth |
| Used by | [[obso]], [[c-obso]], [[drso]], [[xsot\|xOSOT]] | [[space-occupation-gain]], [[expected-value-possession-framework\|EPV]] |

**The validation row is the sharpest difference, and it is not a criticism of either.** Spearman's target — who receives a pass — is directly observable, so it can be tested against outcomes. Fernández & Bornn's target is space quality, and they state plainly that *"there is no existance of ground truth data regarding the quantification of spaces in soccer."* They validate through extensive video review with two FC Barcelona analysts.

One tradition validates against an observable outcome; the other against expert judgement, **because its quantity has no observable ground truth.** See [[pitch-control-traditions-compared]].

## A Shared Motive, Independently Reached

Both origins state low data requirements and reproducibility as design goals. Fernández & Bornn want *"a model that could be applied in a given data frame, without requiring significant data for learning its parameters"*, citing Spanish-league clubs without tracking access. Spearman deliberately minimises frames and needs no ball tracking.

**Both pitch-control traditions were built to be cheap and reproducible**, reached independently — a stronger claim about what the field needed than either paper alone supports.

## The Integration Horizon

PPCF is a differential equation in $T$ and must be integrated to some limit. Both Spearman papers integrate to $T \to \infty$; [[xsot|Yeung & Fujii]] integrate only to **the ball's travel time**, since control gained after the ball would have arrived is useless for shooting. A genuine improvement for **passing-option** use, where the infinite-horizon version over-values distant options.

## What It Is For

Infrastructure rather than an end in itself:

- In [[obso|OBSO]] it appears **twice** — as the control term and inside the transition term raised to a power.
- In [[space-occupation-gain|SOG]] it multiplies [[pitch-value-model|pitch value]] to give quality of owned space.
- In [[xsot|xOSOT]] it discounts each teammate's shot-on-target probability.
- In [[drso|DRSO]] it underpins EF-OBSO on broadcast freeze frames.
- In [[expected-value-possession-framework|the EPV framework]] it defines *pressure* — control below 0.4.

## Not the Same as Off-Ball Value

Control asks *who would win the ball here*; [[off-ball-value]] asks *what would this possession be worth if the ball arrived here*. [[obso|OBSO]] makes the relationship explicit by multiplying them, and [[space-occupation-gain|SOG]] does the same with a different value model.

## Assumptions Worth Knowing

- **Magnus force ignored** in Spearman's trajectory model, so curved passes are mismodelled.
- **Influence additivity** over-states control under overlapping coverage (Gaussian tradition).
- **Instantaneous state.** Neither models where players are *going*; [[trajectory-prediction]] is the natural complement and is combined with neither.
- **Velocity is sometimes unavailable** — [[drso|DRSO]] assumes it and tests five settings.
- **No ground truth for the Gaussian target**, acknowledged by its authors.

## See Also

- [[voronoi-tessellation]] · [[pass-probability-model]] · [[pitch-control-traditions-compared]] · [[tracking-error-propagation]]
- [[obso]] · [[space-occupation-gain]] · [[pitch-value-model]] · [[probability-surface]] · [[off-ball-value]] · [[space-creation]]
- [[c-obso]] · [[drso]] · [[xsot]] · [[expected-possession-value]] · [[soccermap]] · [[theory-based-modelling]] · [[model-selection]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[optical-tracking-data]]
- [[physics-based-pass-probabilities|Spearman 2017]] · [[wide-open-spaces-space-creation|Fernández & Bornn 2018]] · [[beyond-expected-goals|Spearman 2018]]
