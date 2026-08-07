---
title: "OBSO (Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, sports-analytics, pitch-control, probability-surface, optical-tracking-data, player-evaluation, action-valuation, model-decomposition, predictive-validity, construct-validity]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# OBSO (Off-Ball Scoring Opportunity)

[[william-spearman|Spearman's]] (2018) metric for valuing a player who does not have the ball: **if the next on-ball event happened here, how likely is a goal?**

## What It Is Built On

> **Correction, 2026-07-27.** This page previously treated [[pitch-control|PPCF]] as OBSO's control *term*. The dependency runs the other way round. [[physics-based-pass-probabilities|Spearman et al. (2017)]] built a **pass-reception model** a year earlier, validated against who actually received 5,471 held-out passes (81% team, 68% player). OBSO is an *application* of that model — the control term is a pre-existing, independently validated component, not a piece written for this paper.

That matters for how much of OBSO is load-bearing. Its control term arrives already tested against observable outcomes; its transition and score terms do not.

## The Factorisation

$$P(G|D) = \sum_{r} P(S_r \mid C_r, T_r, D)\; P(C_r \mid T_r, D)\; P(T_r \mid D)$$

**The chain rule, exact by construction.** No independence is assumed; each term conditions on the ones after it. Simplification enters only when the score term drops its dependence on $D$, justified because defensive positioning is already proxied through the control term.

| Term | Question | Model | Independently validated? |
|---|---|---|---|
| $T_r$ **Transition** | Where will the next on-ball event occur? | Gaussian **× PPCF$^\alpha$** | No |
| $C_r$ **Control** | Would the attacking team control it there? | [[pitch-control\|PPCF]] | **Yes — 2017 paper** |
| $S_r$ **Score** | Would a goal follow from there? | Distance-only, exponent $\beta$ | No |

Spearman states interpretability as a design goal: each component *answers a specific soccer question* and is independently reusable. The same argument [[structured-model-decomposition|Fernández et al.]] make two years later, reached independently and far more cheaply.

## The Transition Term Is a Decision Model

Displacements between consecutive on-ball events are approximately Gaussian in aggregate. But passers *choose*, preferring passes that will not be intercepted, so the Gaussian is multiplied by attacking pitch control:

$$T(t,\vec r \mid \sigma,\alpha) = N(\vec r, \vec r_b(t), \sigma) \cdot \Big[\sum_{k \in A} PPCF_k(t,\vec r)\Big]^{\alpha}$$

with $\alpha = 1.04$ fitted. The **same PPCF surface appears twice** — as the control term and inside the transition term. That coupling is easy to miss and is what makes the two terms non-independent.

Spearman flags the conspicuous omission himself: **no term prefers passes that move the ball toward goal.** Clear chances are liable to be under-estimated.

## The Score Term Is the Weak One

$S(\vec r|\beta) = [S_d(|\vec r - \vec r_g|)]^{\beta}$, $\beta = 0.48$ fitted. It ignores **angle, defenders and the goalkeeper entirely.** Spearman calls $\beta$ "a fudge factor to ensure that the resultant model can be integrated to give expected scoring".

He also names a bias: the curve is the average scoring chance given *an on-ball event* at that distance, not given a *shot*.

[[c-obso|Teranishi et al.]] replace it with per-degree angular integration discounted by a Gaussian shot-blocking distribution — RMSE 0.309 against 0.324. See [[shot-value-formulations-compared]].

## What It Values, and What It Excludes

**Values:** the player who would *receive* the ball, at the position they occupy.

**Excludes the current ball carrier**, deliberately — his chance was counted at the previous moment.

**Does not value** the player whose movement created the space. OBSO is egocentric; [[c-obso]] fills that gap.

**Does not value a player who will never receive the ball at all**, which is the limitation [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] name explicitly as motivating their framework. OBSO's question is conditional on arrival; a deep midfielder recycling possession is invisible to it.

## Validation

Per-player, match $i$ against match $i{+}1$ across 53 matches:

| Predictor | Next-match goals |
|---|---|
| **OBSO** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**A player's OBSO predicts his next-match goals better than his shots or goals do.** The vault's strongest [[predictive-validity]] evidence: player-level, independent outcome, beating the outcome's own lagged value.

Taken with the 2017 paper, the Spearman line covers **both validation modes** — a component checked against directly observable outcomes (pass receivers), and a derived metric checked against future ones (next-match goals). Few authors here do either.

**This remains the only external-outcome validation of any off-ball metric in the vault**, and its importance grew on 2026-08-07 — see below.

## OBSO as an External Benchmark

> **Added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]], who correlate their [[multi-agent-reinforcement-learning|RL]] Q-values against OBSO at $\rho = -0.305$ on 14 J-League players.

Two things follow, and the second matters more than the number.

**The disagreement is explicable and mild.** OBSO values the receiver; the Q-values reward passers and movers who may never receive. A low negative correlation is close to what the constructions predict. Nakahara et al. make the point cleanly with two players on three season goals each: Miyoshi scores well on OBSO (good movement *to receive*), Theerathon on the Q-values (good passing and other off-ball movement). **Same goal tally, opposite off-ball profiles, and it takes two metrics to see it.**

**OBSO has become the field's reference point.** It is now the only metric here that an *unrelated lineage* has computed and reported alongside its own. That partially weakens the vault's `no-cross-framework-benchmarking` claim on [[action-valuation-frameworks-compared]] — partially, because Nakahara et al. use their own group's reimplementation rather than Spearman's published numbers, and $N = 14$.

Why OBSO and not something else is worth stating: it is **cheap, physically specified, and fully described in its source paper**, so another group can implement it without a licence or a code release. [[vaep|VAEP]] needs a specific event-stream vendor; [[expected-value-possession-framework|Fernández et al.'s EPV]] needs nine neural components and Barcelona's tracking. **Reimplementability, not quality, is what makes a metric a benchmark** — and the metric most often used as a reference here is the one with the weakest score term.

## Descendants

OBSO is the substrate for the entire [[keisuke-fujii|Fujii group]] off-ball and defensive line:

| Work | What it does with OBSO |
|---|---|
| [[c-obso]] | Replaces the score term; differences it against a **predicted trajectory** to credit space creation |
| [[drso]] | Adapts it to broadcast freeze frames (**EF-OBSO**); differences it against an **optimal position** to evaluate defending |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | **Uses it as a comparator, not a substrate** — the first of the group's works to depart from it entirely |

Both C-OBSO and DRSO add a [[counterfactual-baseline|counterfactual]] on top of a value surface — which is why that group's off-ball work sits closer to Spearman's physical tradition than to [[vdep]]'s event classification, despite shared authorship. The RL paper breaks that pattern: no pitch-control model, no value surface, raw positions into a [[gated-recurrent-unit|GRU]].

## The Parameter Chain, Resolved

> **Superseded, 2026-07-27.** This section previously stated that the Fujii-group values "match neither the 2018 fit nor its stated priors", implying they might be wrong. [[physics-based-pass-probabilities|Spearman et al. (2017)]] is now held and settles it.

| Paper | $\sigma$ | $\lambda$ | Provenance |
|---|---|---|---|
| **Spearman et al. (2017)** | **0.45** | **4.30** | Fitted by MLE grid search, with stat *and* syst errors |
| **Spearman (2018)** | **0.54** | **3.99** | Refitted by MAP; priors 0.5 and 4.2 — rounded 2017 values |
| [[c-obso]], [[drso]] | 0.45 | 4.30 | The **2017** values, while citing 2018 |

**The error is bibliographic, not numerical.** The Fujii group uses defensible published fits and attributes them to the wrong paper. The vault's earlier inference — that these were probably the 2017 values — was correct.

Note that [[drso|DRSO]]'s goalkeeper multiplier ($\lambda = 12.9$) appears in **neither** Spearman paper. It is a Fujii-group addition, distinct from the 2018 $\kappa = 1.72$ defensive advantage.

The general lesson stands with a softer edge: **holding a primary source lets a vault audit a literature rather than only summarise it** — but the audit's first finding was less damning than it looked, and holding *both* primaries was needed to see that.

## Corrections to the Earliest Revision

Recorded because the errors were confident and specific:

- **"Factorises under an independence assumption"** — no; the chain rule, exact.
- **"Transition = 2-D Gaussian, σ = 14 m"** — omitted the PPCF$^\alpha$ decision term; 14 m was the *prior*, fitted value **23.9 m**.
- **"PPCF parameters $s = 0.45$, $\lambda = 4.3$"** — right values, wrong paper. See above.
- **Omitted** $\kappa = 1.72$, aerodynamic ball-flight modelling, the offside rule, and the exclusion of the ball carrier.

**Citation-derived pages read as confidently as primary ones and are not.**

## See Also

- [[pitch-control]] · [[c-obso]] · [[drso]] · [[off-ball-value]] · [[space-creation]] · [[probability-surface]]
- [[expected-goals]] · [[shot-value-formulations-compared]] · [[structured-model-decomposition]] · [[predictive-validity]] · [[construct-validity]]
- [[counterfactual-baseline]] · [[multi-agent-reinforcement-learning]] · [[action-space-design]] · [[action-valuation-frameworks-compared]]
- [[william-spearman]] · [[keisuke-fujii]] · [[javier-fernandez]] · [[hiroshi-nakahara]] · [[masakiyo-teranishi]]
- [[physics-based-pass-probabilities|Spearman 2017 Summary]] · [[beyond-expected-goals|Spearman 2018 Summary]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
