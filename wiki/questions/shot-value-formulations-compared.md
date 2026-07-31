---
title: "Are the four shot-value formulations interchangeable?"
type: question
tags: [expected-goals, sports-analytics, model-selection, evaluation, theory-based-modelling, probability-surface, off-ball, needs-review]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.7
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Are the four shot-value formulations interchangeable?

**Status:** Open. One of the six pairwise comparisons exists. The others do not.

> **Correction on framing.** Earlier log entries referred to *three* unbenchmarked xG formulations. Enumerating them properly gives **four**, and the fourth is the one that makes the question interesting.

## The Four

| | Mechanism | Sees defenders? | Sees angle? | Used by |
|---|---|---|---|---|
| **A. Event-data learned** | Boosted / logistic classifier | No | Yes | [[vaep]], [[expected-threat\|xT]]'s recursion, [[pass-carry-reward\|Shelopugin]] |
| **B. Tracking-augmented learned** | Classifier + pressure count, blockage triangle, 3 goalkeeper features | **Yes**, discretely | Yes | [[expected-value-possession-framework\|Fernández et al.]] |
| **C. Distance-only power law** | $[S_d(\lVert \vec r - \vec r_g\rVert)]^{\beta}$, $\beta = 0.48$ | **No** | **No** | [[obso\|OBSO]] as published |
| **D. Angular-geometric physical** | Per-degree integration over shot angle, discounted by defender occlusion | **Yes**, continuously | Yes | [[c-obso]]; refined by [[xsot\|Yeung & Fujii]] |

C is the crudest object in the vault that anyone calls a shot-value model — it does not even use the shooting angle. [[william-spearman|Spearman]] says so himself, calling $\beta$ "a fudge factor to ensure that the resultant model can be integrated to give expected scoring" and proposing replacement.

## The One Comparison That Exists

[[c-obso|Teranishi et al.]] replaced C with D inside OBSO and measured it: **RMSE 0.309 against 0.324** ($p < 10^{-10}$), on 494 shots.

Statistically clear, practically modest — and the qualitative gap is wider than the RMSE suggests, since two shots from equal distance score identically under C and 0.0489 against 0.1202 under D.

That is the only head-to-head anywhere in the vault. A, B and D have never been compared to each other on common data.

## The Observation That Makes This Worth Testing

**[[obso|OBSO]] is the vault's most predictive metric and is built on its worst shot-value model.**

OBSO predicts a player's next-match goals at $r = 0.26$, beating shots (0.17) and goals (0.12) — the strongest [[predictive-validity]] result held here. And it does so using formulation **C**, which ignores angle, defenders and the goalkeeper entirely.

Two readings, and they have opposite implications:

**The score term is not where OBSO's value lies.** If the predictive power comes from the transition and [[pitch-control|control]] terms — from knowing *where the ball will go* and *who will get it* — then the score model is a thin final multiplier and improving it buys little. Under this reading the whole xG literature is optimising a component that is not the bottleneck.

**OBSO is under-performing its own potential.** If a better score term would raise 0.26 further, then a drop-in improvement exists, has existed since 2022 when C-OBSO built it, and nobody has installed it.

These are distinguishable by a single ablation, and nobody has run it.

## What Can Be Settled Analytically

**A cannot be equivalent to B or D.** It is blind to defenders. Two shots with identical location, angle and body part but different occlusion are the same event to A and different events to B and D. Any dataset containing both will separate them.

**B and D model occlusion differently, and the difference is discretisation.** B *counts* defenders inside the shooter-to-posts triangle — an integer. D *integrates* a continuous blocking distribution across the shooting angle. So B cannot distinguish a defender at the edge of the triangle from one central to it; D can. Expect them to agree on clear and heavily-crowded shots, and to diverge on **marginal occlusion** — one defender, partially in the way.

**C is dominated by D** and this is measured, not predicted.

So the analytically interesting comparison is **B against D**, and the prediction is that they disagree on partially-blocked shots specifically.

## Proposed Test

1. **Common-data benchmark.** Fit or compute all four on one shot set. Log loss, Brier, ECE. Establishes whether the differences are large enough to care about.
2. **Stratify by occlusion.** Segment by number of defenders in the shooter-to-posts triangle. Prediction: A degrades monotonically with occlusion; B and D diverge most at exactly one defender.
3. **The decisive ablation.** Recompute [[obso|OBSO]] with each of A, B and D substituted for C, and re-measure next-match-goals correlation against the 0.26 baseline. This answers which of the two readings above is right.
4. **Cost check.** D is computed from geometry with a handful of parameters; A and B need training data. If D matches B on accuracy, it is preferable on every other axis — see [[theory-based-modelling]].

Step 3 is the one worth doing first if only one is possible. Steps 1 and 2 diagnose *why*; step 3 establishes *whether it matters*.

## What Would Change Depending on the Answer

**If OBSO's predictive edge is insensitive to the score term** — then the field's effort on shot-quality modelling is misdirected relative to control and transition modelling, and that is a finding worth publishing on its own. It would also mean [[expected-goals|xG]] improvements have been oversold.

**If it is sensitive** — OBSO should be rebuilt with D, and its 0.26 is a floor rather than a result. Every downstream use ([[c-obso]], the Umemoto defensive line) inherits the improvement.

**If B and D differ materially on marginal occlusion** — then [[expected-value-possession-framework|Fernández et al.'s]] EPV surfaces and the [[keisuke-fujii|Fujii group's]] OBSO-based metrics are not measuring quite the same shot value, which qualifies comparisons between them.

## Why Nobody Has Done It

The same structural reason as [[pitch-control-traditions-compared]]: shot value is a **component**, inherited or replaced within a group rather than benchmarked across groups. Teranishi et al. compared C against D because they were replacing C in their own pipeline — an internal justification, not a survey.

Nobody has an incentive to test whether a competitor's component is better than their own, and component-level differences are invisible in framework-level comparisons. That is the [[action-valuation-frameworks-compared|benchmarking gap]] one level down, and it is worse there, because a shared component that differs silently is harder to notice than two frameworks that differ openly.

## See Also

- [[expected-goals]] · [[obso]] · [[c-obso]] · [[xsot]] · [[pitch-control]]
- [[theory-based-modelling]] · [[predictive-validity]] · [[model-selection]]
- [[pitch-control-traditions-compared]] — the same question about the control term
- [[action-valuation-frameworks-compared]]
- [[beyond-expected-goals|Spearman Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
