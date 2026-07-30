---
title: "OBSO (Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, sports-analytics, pitch-control, probability-surface, optical-tracking-data, player-evaluation, action-valuation, model-decomposition, predictive-validity]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 17%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# OBSO (Off-Ball Scoring Opportunity)

[[william-spearman|Spearman's]] (2018) metric for valuing a player who does not have the ball, by asking: **if the next on-ball event happened here, how likely is a goal?**

*(Rewritten from the [[beyond-expected-goals|primary source]]. The earlier revision was built from citations in Teranishi et al. and contained four errors, listed at the foot of this page.)*

## The Factorisation

$$P(G|D) = \sum_{r} P(S_r \mid C_r, T_r, D)\; P(C_r \mid T_r, D)\; P(T_r \mid D)$$

$D$ is the instantaneous game state — all player positions and velocities.

This is the **chain rule, exact by construction.** No independence is assumed between the three terms; each conditions on the ones after it. Simplification enters only when the score term drops its dependence on $D$, justified on the grounds that defensive positioning is already proxied through the control term.

| Term | Question | Model |
|---|---|---|
| $T_r$ **Transition** | Where will the next on-ball event occur? | Gaussian **× PPCF$^\alpha$**, normalised |
| $C_r$ **Control** | Would the attacking team control it there? | PPCF — see [[pitch-control]] |
| $S_r$ **Score** | Would a goal follow from there? | Distance-only, fitted exponent $\beta$ |

Spearman states interpretability as a design goal: each component *answers a specific soccer question* and is independently reusable. This is the same argument [[structured-model-decomposition|Fernández et al.]] make two years later, reached independently and far more cheaply — three rule-based or lightly-fitted terms against nine trained networks.

## The Transition Term Is a Decision Model

The part most often misdescribed, including previously here.

Displacements between consecutive on-ball events are approximately Gaussian in aggregate — the ball moves by collision with players, so its motion resembles 2-D Brownian motion. But passers *choose*, and they prefer passes that will not be intercepted. So the Gaussian is multiplied by attacking pitch control:

$$T(t,\vec r \mid \sigma,\alpha) = N(\vec r, \vec r_b(t), \sigma) \cdot \Big[\sum_{k \in A} PPCF_k(t,\vec r)\Big]^{\alpha}$$

with $\alpha = 1.04$ fitted — essentially proportional to control. So the *same* PPCF surface appears twice in the model, once as the control term and once inside the transition term. That coupling is easy to miss and is what makes the two terms non-independent.

Spearman flags the conspicuous omission himself: **no term prefers passes that move the ball toward goal.** Chances a coach would read as clear are therefore liable to be under-estimated.

## The Score Term Is the Weak One

$S(\vec r|\beta) = [S_d(|\vec r - \vec r_g|)]^{\beta}$, where $S_d$ is a data-derived scoring curve against distance and $\beta = 0.48$ is fitted.

It ignores **angle, defenders and the goalkeeper entirely.** Spearman calls $\beta$ "a fudge factor to ensure that the resultant model can be integrated to give expected scoring" and proposes replacing it with a proper score model.

He also names a bias in the underlying curve: it is the average scoring chance given *an on-ball event* at that distance, not given a *shot*. An unpressured player is more likely to shoot from 20 m than the average player touching the ball at 20 m — so shot-selection bias is baked in.

This is exactly the term [[c-obso|Teranishi et al.]] replace, with a per-degree angular integration discounted by a Gaussian shot-blocking distribution over goal-side defenders. RMSE 0.309 against 0.324 — a modest improvement, but the qualitative gain is larger, separating shots from equal distance by defender congestion.

## What It Values, and What It Excludes

**Values:** the player who would *receive* the ball, at the position they occupy.

**Excludes the current ball carrier**, deliberately. His scoring chance was counted at the previous moment, before the ball reached him — so OBSO is strictly about players waiting, not the player deciding.

**Does not value** the player whose movement *created* the space for someone else. OBSO is egocentric: what is your position worth *to you*. That gap is what [[c-obso]] fills, and it is why C-OBSO correlates 0.45 with salary on players where plain OBSO correlates −0.28.

## Validation

Per-player, match $i$ against match $i{+}1$ across 53 matches:

| Predictor | Next-match goals |
|---|---|
| **OBSO** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**A player's OBSO predicts his next-match goals better than his shots or goals do.** This is the vault's strongest [[predictive-validity]] evidence: player-level, against an independent outcome, and beating the outcome's own lagged value.

Team level: goals/match against opportunity/match, PCC 0.76 across 14 teams.

## Position in the Vault

| | **OBSO** | [[pitch-control]] | [[probability-surface\|Pass EPV surface]] |
|---|---|---|---|
| Question | What is a goal worth from here, if the ball arrives? | Who would win the ball here? | What is the possession worth if passed here? |
| Estimation | Physical + lightly fitted (6 parameters) | Physical or Gaussian | Learned ([[soccermap]]) |
| Whose value | The receiver's | Neither team's, per se | The possessing team's |
| Cost | Low — ~1,000 frames per match | Low | High |

All three are surfaces read at player positions to produce off-ball value; they differ in what the surface *means*.

OBSO is the substrate for both [[keisuke-fujii|Fujii-group]] counterfactual lines — [[c-obso]] for attacking space creation, and Umemoto & Fujii (2023) for defensive positioning — which places that work closer to Spearman's physical tradition than to [[vdep]]'s event classification, despite the shared authorship.

## Corrections to the Earlier Revision

Recorded because the errors were confident and specific, which is the dangerous kind:

- **"Factorises under an independence assumption"** — no; it is the chain rule, exact.
- **"Transition = 2-D Gaussian, σ = 14 m"** — omitted the PPCF$^\alpha$ decision term entirely, and 14 m was the *prior*, taken from observed displacement spread. The fitted value is **23.9 m**.
- **"PPCF parameters $s = 0.45$, $\lambda = 4.3$"** — this paper fits **$s = 0.54$, $\lambda = 3.99$**, with priors of 0.5 and 4.2 drawn from Spearman et al. (2017). The previously recorded values came via Teranishi and match neither exactly.
- **Omitted** the $\kappa = 1.72$ defensive advantage, aerodynamic ball-flight modelling, the offside rule, and the exclusion of the ball carrier.

The general lesson, which the vault has now recorded twice in different forms: **citation-derived pages read as confidently as primary ones and are not.** Provenance warnings help; acquiring the source is the only real fix.

## See Also

- [[c-obso]] · [[pitch-control]] · [[off-ball-value]] · [[space-creation]] · [[probability-surface]]
- [[expected-goals]] · [[structured-model-decomposition]] · [[predictive-validity]] · [[optical-tracking-data]]
- [[william-spearman]] · [[keisuke-fujii]] · [[javier-fernandez]]
- [[beyond-expected-goals|Source Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
