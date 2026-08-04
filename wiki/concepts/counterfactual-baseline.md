---
title: "Counterfactual Baseline"
type: concept
tags: [counterfactual, evaluation, machine-learning, sports-analytics, player-evaluation, trajectory-prediction, policy-modelling, statistics]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 40%
  inferred: 38%
  generated: 17%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Counterfactual Baseline

Evaluating an agent by the **difference between what actually happened and what a model says should have happened**. The model supplies a reference; the deviation is the credit.

A recurring pattern across this vault's sports sources, usually presented as a domain trick, and general enough to name.^[generated: no source names this pattern or groups these instances. rests-on: source:cobso-construction, source:drso-construction, source:epva-league-average]

## The Shape

1. Fit or specify a model of *reference* behaviour — a league-average player, an expected trajectory, an optimal position.
2. Compute the outcome under that reference.
3. Compute the outcome that actually occurred.
4. **Attribute the difference to the agent.**

The move solves a specific problem: raw outcome values are dominated by situation rather than by the agent. A striker in the six-yard box has a high scoring probability regardless of who he is. Differencing against a reference removes the situation.

## Instances Here

| Framework | Reference | Deviation attributed to |
|---|---|---|
| [[martingale-epv\|EPVA]] | A hypothetical **league-average player** | The actual player's touches |
| [[c-obso]] | A **predicted trajectory** from GVRNN trained on opponents | The off-ball player's movement |
| **[[drso\|DRSO]]** | The **best reachable grid vertex** for that defender | The defender's positioning |
| [[policy-modelling\|Fernández et al.]] | The **best available pass** on the surface | The decision actually taken |

The four differ in what the reference *is*, and the choice matters more than the machinery:^[generated: the three-way taxonomy of reference types is constructed here. rests-on: source:epva-league-average, source:cobso-construction, source:drso-construction]

- **A population average** (EPVA) asks "better than a typical player?"
- **A predicted behaviour** (C-OBSO) asks "different from what was expected here?"
- **An optimum** (DRSO, Fernández et al.) asks "how far from the best available?"

These are not interchangeable. A player who consistently does the predictable-but-excellent thing scores zero on the second and highly on the first and third.

Note that [[c-obso]] and [[drso]] come from the **same research group** and sit at opposite ends of that axis — one measures deviation from *normal*, the other from *optimal*. The choice is deliberate rather than incidental.

## Why Counterfactual Baselines Individuate

> ### `counterfactual-individuates`
>
> **The individuating ingredient is the counterfactual, not the data.**
>
> ^[generated: the vault's own diagnosis, drawn from comparing VDEP, GVDEP, C-OBSO and DRSO. No source states it. The most widely propagated generated claim in this vault — also on [[off-ball-value]], [[defensive-valuation]] and the synthesis. rests-on: source:vdep-team-level, source:gvdep-team-level, source:cobso-per-player, source:drso-per-defender]

[[vdep]] and [[gvdep]] put player positions into one classifier and get **one number for the configuration**, with no principled way to split it — which is why both are team-level. [[c-obso]] and [[drso]] use comparable data and produce per-player numbers.

The difference is not the data or the model. It is that the counterfactual **intervenes on one named agent**. Once you ask "what if *this* player had moved differently", the credit is unambiguously his.

**Supporting evidence, 2026-07-27.** [[drso|DRSO]] is a direct test. Same research group, same domain, comparable input data — and it computes $Diff_{opt-obs}$ **for each named defender**, precisely because it intervenes on one defender's position at a time. The prediction held.

**What would still falsify it.** The claim says intervention is *necessary* for per-agent credit, not merely sufficient. A counter-example would be a method that individuates by **attribution** alone — a Shapley-style decomposition over players, assigning per-agent credit from a collective model without intervening on anyone. [[shap]] does exactly that for *features*, and nothing in principle prevents the same over agents.

**Status:** supported on four instances, not proven. Sufficiency is well evidenced; necessity is not. Treat as a strong tendency.

## The Dependency Problem

**A fitted reference model passes its errors into the metric.**

[[c-obso]] states the sharpest version: if trajectory prediction were perfect, C-OBSO would be identically zero. Consequences:

- **Values are not portable.** A different predictor gives a different metric.
- **Improvement destroys the measurement.** Better prediction shrinks the signal.
- **Predictor error masquerades as agent behaviour.** C-OBSO clips negatives to zero precisely because the *predicted defender* often behaves implausibly.

**Optimum-referenced baselines escape this** and pay differently. [[drso|DRSO]] has no fitted predictor to be wrong — its reference is a search over candidate positions. But it inherits a different problem: **the optimum may be unreachable.** DRSO never checks whether the defender could physically have got to the winning vertex, which its authors name as a limitation.

So the trade is: predicted references risk *model error*, optimum references risk *infeasibility*. EPVA's hypothetical league-average player avoids both and pays in extrapolation to a player who does not exist.

## Relation to Counterfactual Simulation

| | **Counterfactual baseline** | [[counterfactual-simulation\|Counterfactual simulation]] |
|---|---|---|
| Intervention | Replace behaviour or position with a *reference* | Replace an *entity* with a different one |
| Question | How much did this agent deviate, and did it help? | What would a different agent have produced? |
| Output | Credit for the observed agent | Projection for an unobserved one |
| Examples | [[c-obso]], [[drso]], EPVA | [[scoutgpt]], [[eventgpt]] |

Simulation answers recruitment questions; baselines answer valuation questions.

## Beyond Sport

The pattern is standard wherever performance must be separated from circumstance: risk-adjusted clinical outcomes, value-added models in education, benchmark-relative returns in finance.^[imported: background knowledge, not from any held source] Disputes about the metric are usually disputes about the reference.

## See Also

- [[c-obso]] · [[drso]] · [[trajectory-prediction]] · [[martingale-epv]] · [[off-ball-value]] · [[imitation-learning]]
- [[counterfactual-simulation]] · [[policy-modelling]] · [[probability-surface]] · [[shap]]
- [[defensive-valuation]] · [[vdep]] · [[gvdep]] · [[action-valuation]] · [[obso]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
