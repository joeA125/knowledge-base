---
title: "Beyond Expected Goals (Spearman, 2018)"
type: source_summary
tags: [summary, sports-analytics, off-ball, pitch-control, optical-tracking-data, probability-surface, predictive-validity, model-decomposition, bayesian, player-evaluation, recruitment]
sources: [raw/papers/beyond_expected_goals.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 12%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Beyond Expected Goals

**Author:** [[william-spearman]] — **Hudl**
**Venue:** MIT Sloan Sports Analytics Conference, 2018 Research Papers Competition.

The source flagged as acquisition priority 1 across four log entries. [[obso|OBSO]] and [[pitch-control|PPCF]] have been dependencies of five vault pages, described entirely from citations in [[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] and [[expected-value-possession-framework|Fernández et al.]] Reading it primary corrects four errors — see [Corrections](#corrections-to-the-vaults-second-hand-account).

## The Question

> How do you value a player standing unmarked at the far post waiting for a cross that never arrives?

Both of the paper's opening examples are of **opportunities created but never realised**: the tall centre forward at the far post, and the winger played onside whose run is wasted because the pass never comes. In each case the attacker has done something valuable and generated no event.

## The Model

The probability that the attacking team scores **with the next on-ball event**, summed over pitch locations:

$$P(G|D) = \sum_{r} P(S_r \mid C_r, T_r, D)\; P(C_r \mid T_r, D)\; P(T_r \mid D)$$

$D$ is the instantaneous game state — all player positions and velocities. The three terms answer three soccer questions:

| Term | Question | Model |
|---|---|---|
| $T_r$ **Transition** | Where will the next on-ball event occur? | Gaussian × PPCF$^\alpha$, normalised |
| $C_r$ **Control** | Would the attacking team control it there? | **PPCF** — potential pitch control field |
| $S_r$ **Score** | Would a goal follow from there? | Distance-only, with a fitted exponent |

This is a **chain-rule decomposition, exact by construction** — not an independence assumption. Simplification enters later, when the score term drops its dependence on $D$ on the argument that defensive positioning is already proxied through PPCF.

Spearman lists interpretability as a design goal in its own right: *each component of the model has real soccer meaning and answers a specific soccer question.* Compare [[structured-model-decomposition]] — the same argument Fernández et al. make two years later, reached independently and far more cheaply.

## PPCF: Control as a Poisson Process

The paper's substantive machinery, extending Spearman et al. (2017).

$$\frac{dPPCF_j}{dT} = \Big(1 - \sum_k PPCF_k\Big)\, f_j(t,\vec r,T|s)\, \lambda_j$$

A player near the ball uncontested becomes progressively more likely to make a controlled touch. The leading bracket makes control **zero-sum**: probability mass gained by one player is removed from everyone else.

$f_j$ is the probability player $j$ reaches $r$ within time $T$ — a **logistic** CDF over the residual between expected and true intercept time. Logistic is chosen over normal deliberately, *for its heavier tails*, absorbing tracking error, player facing, awareness and tactical decisions without modelling any of them. Expected intercept time assumes constant acceleration 7 m/s² to a maximum 5 m/s.

Three refinements over the 2017 model:

**Ball flight time.** Passes are not instantaneous. Trajectories are simulated with **aerodynamic drag** (Asai & Seo, 2013), and $PPCF_j$ is held at zero until the ball could physically arrive — so longer passes give both teams more time to converge. The flight time chosen is the one **most advantageous to the attackers**, an explicit thumb on the scale that Spearman says compensates for weaknesses in the decision model.

**Defensive advantage.** A second parameter $\kappa$ scales the control rate for defenders, on the reasoning that a defender is satisfied with heading clear while an attacker needs a controlled touch. Fitted at **1.72** — a substantial asymmetry, and one no other pitch-control model in this vault includes.

**Offside.** $\lambda_i$ is set to zero for attackers in offside positions.

## Transition: A Decision Model, Not Just a Gaussian

Displacements between consecutive on-ball events are approximately Gaussian in aggregate — the ball moves by collision with players, so its motion resembles 2-D Brownian motion.

But passers choose. So the transition density multiplies the Gaussian by attacking pitch control raised to a power:

$$T(t,\vec r \mid \sigma,\alpha) = N(\vec r, \vec r_b(t), \sigma) \cdot \Big[\sum_{k \in A} PPCF_k(t,\vec r)\Big]^{\alpha}$$

normalised to unity. $\alpha$ scales how strongly passers prefer retaining possession; fitted at **1.04**, essentially proportional.

Spearman flags the conspicuous omission himself: **there is no term preferring passes that move the ball toward goal.** Consequently chances a coach would read as clear may be *under*-estimated.

## Parameters

Fitted by MAP with normally distributed priors on 5 training matches, Bayesian rather than ML because event–tracking synchronisation carries ~3 s of noise.

| Parameter | MAP | Meaning |
|---|---|---|
| $s$ | **0.54** s | Temporal uncertainty on intercept time |
| $\lambda$ | **3.99** Hz | Control rate |
| $\kappa$ | **1.72** | Defensive advantage |
| $\sigma$ | **23.9** m | Transition spread |
| $\alpha$ | **1.04** | Preference for retaining possession |
| $\beta$ | **0.48** | Score-model exponent |

$\beta < 1$ flattens the distance–scoring curve, improving chances further from goal. Spearman calls it "a fudge factor to ensure that the resultant model can be integrated to give expected scoring" and proposes replacing it with a better score model.

## Validation — The Result That Matters

The stated objective is **a leading indicator less stochastic than scoring itself**. Per-player, correlating match $i$ against match $i{+}1$ across 53 test matches:

| Predictor ↓ / Outcome → | Next-match OBSO | Next-match shots | **Next-match goals** |
|---|---|---|---|
| **OBSO** | 0.60 | 0.37 | **0.26** |
| Shots | ~ | 0.35 | 0.17 |
| Goals | ~ | ~ | 0.12 |

**OBSO predicts a player's next-match goals better than his shots or his goals do.** Goals are the worst predictor of future goals — the vault's recurring finding, here at **player level against an independent outcome**, which is stronger than anything else it holds. See [[predictive-validity]].

Team-level: goals/match against opportunity/match across 14 teams, PCC **0.76**.

## Applications

**Tactical moment analysis.** Integrated opportunity magnitude ranks moments for video review; the maps explain each one. Directly targets analyst hours.

**Match analysis.** Team I outshot Team D heavily, yet integrated opportunity was near-identical (1.32 vs 1.47) — matching the 1–1 scoreline. Shot count misled; opportunity did not.

**Team performance.** Four low-ranked teams cluster below the conversion trend, attributed to passing/receiving skill, opportunity awareness, or finishing.

**Talent identification.** Ranking by mean OBSO/match surfaces mostly centre forwards, but also an attacking right-back and a Team M midfielder with **zero goals** whose opportunity creation suggests latent threat. Per-player opportunity maps stay stable across matches while shots and goals fluctuate — the metric is measuring something persistent about positioning.

## Reproducibility as a Design Constraint

Unusual and worth borrowing. Spearman deliberately minimises data requirements so results transfer across providers:

- **No ball tracking needed** — on-ball events identify who is touching it.
- **One frame per event, not 25 Hz** — ~1,000 frames per match rather than ~8,000,000.
- Only three event fields required: when, who, and whether it was a goal/shot/other.

Contrast [[martingale-epv|Cervone et al.'s]] 461 processors. This is the same off-ball question answered at a fraction of the cost, and the constraint is a modelling *choice* rather than a limitation.

## Corrections to the Vault's Second-Hand Account

| Vault previously said | Actually |
|---|---|
| Factorises "under an independence assumption" | **Chain rule, exact.** Simplification enters only in the score term |
| Transition = "2-D Gaussian, σ = 14 m" | Gaussian **× PPCF$^\alpha$** — a decision model. σ is fitted at **23.9 m**; 14 m was the prior, from observed displacement spread |
| PPCF parameters $s = 0.45$, $\lambda = 4.3$ | This paper fits **$s = 0.54$, $\lambda = 3.99$**, with priors 0.5 and 4.2 taken from Spearman et al. (2017). The values previously recorded came via Teranishi et al. and match neither exactly — most likely the 2017 published fit |
| Spearman "associated with Liverpool FC" | **Hudl**, at time of writing. The Liverpool association was inferred from wider literature and is not supported by any held source |

Also absent from the vault's account entirely: the **$\kappa = 1.72$ defensive advantage**, **aerodynamic ball-flight modelling**, the **offside rule**, and the fact that OBSO **excludes the current ball carrier** — his scoring chance was counted at the previous moment, before the ball reached him.

## Assessment

**Strengths.** Every component is independently meaningful and independently reusable, which Spearman says explicitly is the point. The validation is the cleanest in the vault's football coverage: a player-level leading indicator beating both shots and goals at predicting goals. Physical grounding — drag, acceleration limits, arrival-time contests — means parameters have units and priors can come from measurement rather than taste. And the reproducibility constraints are a deliberate design choice, not a concession.

**Weaknesses, most acknowledged by the author.**

- **No forward-progress term** in the transition model, so obvious chances are under-valued.
- **$\beta$ is admitted to be a fudge factor**, and the score model ignores angle, defenders and the keeper entirely — the weakest of the three components by some distance, and the one [[c-obso|Teranishi et al.]] later replaced.
- **Crossing difficulty while running at speed is unmodelled**, over-valuing chances created on the run.
- **Shot-selection bias** in the data-derived score curve: a touch 20 m from goal is not a shot from 20 m.
- **58 matches, one anonymised 14-team league, one season.** Only 53 in test.
- **No comparison against any competing model** — not xG, not dangerosity, not Lucey et al. Consistent with the vault-wide benchmarking gap.
- Blocked passes are not modelled, handled only by proxy through defender control.

## See Also

- [[obso]] · [[pitch-control]] · [[off-ball-value]] · [[space-creation]] · [[c-obso]]
- [[probability-surface]] · [[expected-goals]] · [[predictive-validity]] · [[structured-model-decomposition]]
- [[william-spearman]] · [[optical-tracking-data]] · [[recruitment]]
- [[action-valuation-frameworks-compared]]
