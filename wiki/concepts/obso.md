---
title: "OBSO (Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, sports-analytics, pitch-control, probability-surface, optical-tracking-data, player-evaluation, action-valuation, model-decomposition, predictive-validity]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/team_defense_positioning_statsbomb.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# OBSO (Off-Ball Scoring Opportunity)

[[william-spearman|Spearman's]] (2018) metric for valuing a player who does not have the ball, by asking: **if the next on-ball event happened here, how likely is a goal?**

*(Rewritten from the [[beyond-expected-goals|primary source]]. The earlier revision, built from citations, contained four errors — listed at the foot.)*

## The Factorisation

$$P(G|D) = \sum_{r} P(S_r \mid C_r, T_r, D)\; P(C_r \mid T_r, D)\; P(T_r \mid D)$$

$D$ is the instantaneous game state — all player positions and velocities.

This is the **chain rule, exact by construction.** No independence is assumed; each term conditions on the ones after it. Simplification enters only when the score term drops its dependence on $D$, justified because defensive positioning is already proxied through the control term.

| Term | Question | Model |
|---|---|---|
| $T_r$ **Transition** | Where will the next on-ball event occur? | Gaussian **× PPCF$^\alpha$**, normalised |
| $C_r$ **Control** | Would the attacking team control it there? | PPCF — see [[pitch-control]] |
| $S_r$ **Score** | Would a goal follow from there? | Distance-only, fitted exponent $\beta$ |

Spearman states interpretability as a design goal: each component *answers a specific soccer question* and is independently reusable. The same argument [[structured-model-decomposition|Fernández et al.]] make two years later, reached independently and far more cheaply — three lightly-fitted terms against nine trained networks.

## The Transition Term Is a Decision Model

The part most often misdescribed, including previously here.

Displacements between consecutive on-ball events are approximately Gaussian in aggregate. But passers *choose*, and prefer passes that will not be intercepted. So the Gaussian is multiplied by attacking pitch control:

$$T(t,\vec r \mid \sigma,\alpha) = N(\vec r, \vec r_b(t), \sigma) \cdot \Big[\sum_{k \in A} PPCF_k(t,\vec r)\Big]^{\alpha}$$

with $\alpha = 1.04$ fitted. So the **same PPCF surface appears twice** in the model — once as the control term, once inside the transition term. That coupling is easy to miss and is what makes the two terms non-independent.

Spearman flags the conspicuous omission himself: **no term prefers passes that move the ball toward goal.** Clear chances are therefore liable to be under-estimated.

## The Score Term Is the Weak One

$S(\vec r|\beta) = [S_d(|\vec r - \vec r_g|)]^{\beta}$, with $\beta = 0.48$ fitted. It ignores **angle, defenders and the goalkeeper entirely.** Spearman calls $\beta$ "a fudge factor to ensure that the resultant model can be integrated to give expected scoring".

He also names a bias: the curve is the average scoring chance given *an on-ball event* at that distance, not given a *shot*. An unpressured player is likelier to shoot from 20 m than the average player touching the ball there.

[[c-obso|Teranishi et al.]] replace it with per-degree angular integration discounted by a Gaussian shot-blocking distribution — RMSE 0.309 against 0.324. See [[shot-value-formulations-compared]].

## What It Values, and What It Excludes

**Values:** the player who would *receive* the ball, at the position they occupy.

**Excludes the current ball carrier**, deliberately — his chance was counted at the previous moment.

**Does not value** the player whose movement created the space. OBSO is egocentric. That gap is what [[c-obso]] fills.

## Validation

Per-player, match $i$ against match $i{+}1$ across 53 matches:

| Predictor | Next-match goals |
|---|---|
| **OBSO** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**A player's OBSO predicts his next-match goals better than his shots or goals do.** The vault's strongest [[predictive-validity]] evidence: player-level, independent outcome, beating the outcome's own lagged value.

## Descendants

OBSO is the substrate for the entire [[keisuke-fujii|Fujii group]] off-ball and defensive line:

| Work | What it does with OBSO |
|---|---|
| [[c-obso]] | Replaces the score term; differences it against a **predicted trajectory** to credit space creation |
| [[drso]] | Adapts it to incomplete broadcast data (**EF-OBSO**); differences it against an **optimal position** to evaluate defending |

Both use OBSO as a value surface and add a [[counterfactual-baseline|counterfactual]] on top — which is why that group's off-ball work sits closer to Spearman's physical tradition than to [[vdep]]'s event classification, despite shared authorship.

## ⚠️ A Propagated Parameter Error

Spearman (2018) fits the PPCF parameters at **$s = 0.54$ and $\lambda = 3.99$**, by MAP with priors of 0.5 and 4.2 taken from Spearman et al. (2017).

**Two Fujii-group papers use $\sigma = 0.45$ and $\lambda = 4.3$ while citing Spearman (2018):**

- [[c-obso|Teranishi et al. (2022/23)]]
- [[drso|Umemoto & Fujii (2023)]] — explicitly "following the previous study [6]", where [6] is Spearman (2018)

The values match neither the 2018 fit nor its stated priors. The most likely explanation remains that they are the **2017 published fit**, transcribed forward through a research line.

This vault recorded the wrong values too, from the same secondary route, until the primary source was acquired. The general point: **holding a primary source lets a vault audit a literature rather than only summarise it** — this discrepancy is invisible to anyone reading only the descendants.

Note also that Umemoto & Fujii add a **goalkeeper multiplier** ($\lambda = 12.9$, three times outfield). That is new, and is *not* Spearman's $\kappa = 1.72$ defensive advantage — a keeper-specific control rate rather than a general defender one.

Acquiring Spearman et al. (2017) would settle whether 0.45/4.3 are its fitted values.

## Corrections to the Earlier Revision

Recorded because the errors were confident and specific:

- **"Factorises under an independence assumption"** — no; it is the chain rule, exact.
- **"Transition = 2-D Gaussian, σ = 14 m"** — omitted the PPCF$^\alpha$ decision term, and 14 m was the *prior*. Fitted value **23.9 m**.
- **"PPCF parameters $s = 0.45$, $\lambda = 4.3$"** — see above.
- **Omitted** the $\kappa = 1.72$ defensive advantage, aerodynamic ball-flight modelling, the offside rule, and the exclusion of the ball carrier.

**Citation-derived pages read as confidently as primary ones and are not.** Provenance warnings help; acquiring the source is the only real fix.

## See Also

- [[c-obso]] · [[drso]] · [[pitch-control]] · [[off-ball-value]] · [[space-creation]] · [[probability-surface]]
- [[expected-goals]] · [[shot-value-formulations-compared]] · [[structured-model-decomposition]] · [[predictive-validity]]
- [[counterfactual-baseline]] · [[william-spearman]] · [[keisuke-fujii]] · [[javier-fernandez]]
- [[beyond-expected-goals|Source Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
