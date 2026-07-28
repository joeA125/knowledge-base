---
title: "Counterfactual Baseline"
type: concept
tags: [counterfactual, evaluation, machine-learning, sports-analytics, player-evaluation, trajectory-prediction, policy-modelling, statistics]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.8
provenance:
  extracted: 40%
  inferred: 55%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Counterfactual Baseline

Evaluating an agent by the **difference between what actually happened and what a predictive model says should have happened**. The model supplies a reference; the deviation is the credit.

A recurring pattern across this vault's sports sources, usually presented as a domain trick, and general enough to name.

## The Shape

1. Fit a model of *typical* behaviour — a league-average player, an expected trajectory, an average policy.
2. Compute the outcome under that reference.
3. Compute the outcome that actually occurred.
4. **Attribute the difference to the agent.**

The move solves a specific problem: raw outcome values are dominated by situation rather than by the agent. A striker in the six-yard box has a high scoring probability regardless of who he is. Differencing against a reference removes the situation and leaves what the agent contributed to it.

## Instances Here

| Framework | Reference | Deviation attributed to |
|---|---|---|
| [[martingale-epv\|EPVA]] | A hypothetical **league-average player** in the same situation | The actual player's touches |
| [[c-obso]] | A **predicted trajectory** from GVRNN trained on opponents | The off-ball player's movement |
| Umemoto & Fujii (2023) | The **best alternative grid cell** a defender could occupy | The defender's positioning |
| [[policy-modelling\|Fernández et al.]] | The **best available pass** on the surface | The decision actually taken |

The four differ in what the reference *is*, and the choice matters more than the machinery:

- **A population average** (EPVA) asks "better than a typical player?"
- **A predicted behaviour** (C-OBSO) asks "different from what was expected here?"
- **An optimum** (the other two) asks "how far from the best available?"

These are not interchangeable. Deviating from expectation is not the same as exceeding the average, and neither is the same as approaching optimal. A player who consistently does the predictable-but-excellent thing scores zero on the second and highly on the first and third.

## Why Counterfactual Baselines Individuate

The property that makes this pattern valuable, and it is easy to miss.

[[vdep]] puts all 22 player positions into one classifier and gets **one number for the configuration**, with no principled way to split it — which is why it is team-level. [[c-obso]] uses the *same kind of data* and produces an individual number.

The difference is not the data or the model. It is that the counterfactual **intervenes on one named agent**. Once you ask "what if *this* player had moved differently", the credit is unambiguously his. Intervention is the individuating ingredient, and it is available to any framework willing to pay for a reference model.

That generalises: wherever collective data resists per-agent attribution, a counterfactual on one agent is the standard route out.

## The Dependency Problem

The weakness is structural and shared. **The metric inherits the reference model's errors and cannot outlive its improvement.**

[[c-obso]] states the sharpest version: if trajectory prediction were perfect, C-OBSO would be identically zero. The metric requires its own reference to be wrong. Consequences:

- **Values are not portable.** A different predictor gives a different metric, so cross-study comparison is meaningless without fixing the reference.
- **Improvement destroys the measurement.** Better prediction shrinks the signal.
- **Predictor error masquerades as agent behaviour.** C-OBSO clips negative values to zero precisely because the *predicted defender* often behaves implausibly — an admission that some of the measured deviation is model failure, not player action.

EPVA avoids this by using a *hypothetical* league-average player rather than a fitted predictor, but pays a different price: the baseline is a heavy extrapolation to a player who does not exist.

## Relation to Counterfactual Simulation

Distinct, and the vault holds both.

| | **Counterfactual baseline** | [[counterfactual-simulation\|Counterfactual simulation]] |
|---|---|---|
| Intervention | Replace behaviour with a *predicted reference* | Replace an *entity* with a different one |
| Question | How much did this agent deviate, and did it help? | What would a different agent have produced? |
| Output | Credit for the observed agent | Projection for an unobserved one |
| Examples | [[c-obso]], EPVA | [[scoutgpt]], [[eventgpt]] |

Simulation answers recruitment questions; baselines answer valuation questions. Both need a generative or predictive model, and both inherit its biases — see the causal caveats on [[counterfactual-simulation]], which apply here unchanged.

## Beyond Sport

The pattern is standard wherever performance must be separated from circumstance: risk-adjusted clinical outcomes (observed minus predicted mortality given case mix), value-added models in education (pupil progress against predicted progress), and benchmark-relative returns in finance. Each faces the same dependency — the adjustment is only as good as the model producing the expectation, and disputes about the metric are usually disputes about the reference.

## See Also

- [[c-obso]] · [[trajectory-prediction]] · [[martingale-epv]] · [[off-ball-value]]
- [[counterfactual-simulation]] · [[policy-modelling]] · [[probability-surface]]
- [[defensive-valuation]] · [[vdep]] · [[action-valuation]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
