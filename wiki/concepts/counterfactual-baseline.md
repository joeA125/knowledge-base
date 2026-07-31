---
title: "Counterfactual Baseline"
type: concept
tags: [counterfactual, evaluation, machine-learning, sports-analytics, player-evaluation, trajectory-prediction, policy-modelling, statistics]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 40%
  generated: 20%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Counterfactual Baseline

Evaluating an agent by the **difference between what actually happened and what a predictive model says should have happened**. The model supplies a reference; the deviation is the credit.

A recurring pattern across this vault's sports sources, usually presented as a domain trick, and general enough to name.^[generated: no source names this pattern or groups these instances; the abstraction is constructed here]

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

The four differ in what the reference *is*, and the choice matters more than the machinery:^[generated: the three-way taxonomy of reference types is constructed here]

- **A population average** (EPVA) asks "better than a typical player?"
- **A predicted behaviour** (C-OBSO) asks "different from what was expected here?"
- **An optimum** (the other two) asks "how far from the best available?"

These are not interchangeable. Deviating from expectation is not the same as exceeding the average, and neither is the same as approaching optimal. A player who consistently does the predictable-but-excellent thing scores zero on the second and highly on the first and third.

## Why Counterfactual Baselines Individuate

> ⚠️ ^[generated: this is the vault's own diagnosis, drawn from comparing VDEP and C-OBSO. Neither paper states it, and it is the most widely propagated generated claim in this vault — it also appears on [[off-ball-value]], [[defensive-valuation]] and the synthesis. Untested.]

The property that makes this pattern valuable, and it is easy to miss.

[[vdep]] puts all 22 player positions into one classifier and gets **one number for the configuration**, with no principled way to split it — which is why it is team-level. [[c-obso]] uses the *same kind of data* and produces an individual number.

The difference is not the data or the model. It is that the counterfactual **intervenes on one named agent**. Once you ask "what if *this* player had moved differently", the credit is unambiguously his.

**What would falsify it.** The claim predicts that any framework producing per-agent credit from collective data does so via an intervention, and that non-interventional methods cannot. A counter-example would be a method that individuates by attribution alone — a Shapley-style decomposition over players, say, which assigns per-agent credit from a collective model without intervening on anyone. [[shap]] does exactly that for *features*, and nothing in principle prevents the same over agents. So the claim may be a description of what this literature happens to do rather than a necessity.

## The Dependency Problem

The weakness is structural and shared. **The metric inherits the reference model's errors and cannot outlive its improvement.**

[[c-obso]] states the sharpest version: if trajectory prediction were perfect, C-OBSO would be identically zero. The metric requires its own reference to be wrong. Consequences:

- **Values are not portable.** A different predictor gives a different metric.
- **Improvement destroys the measurement.** Better prediction shrinks the signal.
- **Predictor error masquerades as agent behaviour.** C-OBSO clips negative values to zero precisely because the *predicted defender* often behaves implausibly.

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

The pattern is standard wherever performance must be separated from circumstance: risk-adjusted clinical outcomes (observed minus predicted mortality given case mix), value-added models in education, and benchmark-relative returns in finance.^[imported: these parallels are background knowledge, not drawn from any held source] Each faces the same dependency — the adjustment is only as good as the model producing the expectation, and disputes about the metric are usually disputes about the reference.

## See Also

- [[c-obso]] · [[trajectory-prediction]] · [[martingale-epv]] · [[off-ball-value]] · [[imitation-learning]]
- [[counterfactual-simulation]] · [[policy-modelling]] · [[probability-surface]] · [[shap]]
- [[defensive-valuation]] · [[vdep]] · [[action-valuation]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
