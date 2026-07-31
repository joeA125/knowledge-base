---
title: "Do players decide suboptimally, or do the models only think so?"
type: question
tags: [policy-modelling, game-theory, action-valuation, evaluation, predictive-validity, counterfactual, sports-analytics, needs-review]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.65
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Do players decide suboptimally, or do the models only think so?

**Status:** Open, and more open than earlier vault entries implied.

Two frameworks with nothing methodologically in common each report a large gap between what players do and what their model says is best. Neither cites the other.

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value [[probability-surface\|surface]] | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | [[game-theory\|Game-theoretic]] payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |

## First: They Are Not the Same Claim

Earlier vault entries described these as "both locating the divergence in the same place." That was too strong, and the distinction matters.

**Fernández et al. measure suboptimal *targeting within* an action.** Given that a pass is made, it often does not go to the highest-value destination on the surface. The action type is not in question.

**Yeung & Fujii measure suboptimal *selection between* actions.** Given a shot-taking situation, the shooter shoots when the equilibrium says pass. The destination is not in question.

So "players shoot too much" is Yeung & Fujii's claim alone. What the two jointly support is weaker and still substantial: **a large observed-versus-optimal gap appears under two unrelated methods, on different leagues, different data types, and different decision granularities.**

That convergence is worth taking seriously precisely because the methods share no machinery — but it is convergence on the *existence* of a gap, not on its cause.

## The Objection That Has To Be Answered First

Both gaps are measured **by the model's own value function**. "Suboptimal by the model's lights" is a statement about internal consistency, not about football.

Three specific ways the gap could be an artefact:

**1. Average-player models applied to specific players.** [[expected-goals|xG]], [[xsot|xSOT]] and EPV are deliberately player-agnostic — see the feature-exclusion discussion on [[expected-goals]]. The optimal action for an *average* player is not the optimal action for an elite finisher. A striker with genuine finishing skill *should* shoot more than an average-player equilibrium prescribes, and would be scored as over-shooting for doing the right thing.

**2. Execution difficulty is unmodelled.** Both frameworks value the *intent* of the best available option, not the difficulty of executing it. A cutback that is worth 0.112 in expectation may be substantially harder to play than the safe ball worth 0.032. This is precisely the [[intent-vs-outcome-valuation|intent/outcome distinction]] — and it means the "gap" may be the price of reliability rather than an error.

**3. Equilibrium assumes a rational opponent.** Yeung & Fujii's Nash solution assumes the defender is also best-responding. If defenders systematically fail to block when they should, the shooter's true best response shifts toward shooting, and the observed behaviour may be closer to correct than the equilibrium suggests.

Any of the three could produce a large gap with no decision error present at all.

## What Can Be Settled From Held Sources

Very little, which is why this is a question rather than a finding.

What *is* established: the gap is large under both methods, and it is not obviously an artefact of one method's peculiarities, since the methods have no peculiarities in common.

What is not established: whether the gap tracks anything outside the models. Neither paper tests this. Yeung & Fujii report an aggregate equilibrium pooled across all shooters and defenders; Fernández et al. report the gap as an illustrative frame in a control-room application.

## Proposed Test

The question is one of **prescriptive validity** — does deviation from the model's optimum predict worse real outcomes? — and nothing in this vault tests it.

1. **Does the gap predict anything?** Compute per-player or per-team gap over a season and correlate with goals, points, or [[predictive-validity|next-match]] outcomes. If a larger gap predicts nothing, the models are self-consistent and uninformative about behaviour. This is the decisive step and the cheapest.
2. **Condition on player skill.** Re-run the equilibrium with shooter-specific finishing ability, which Yeung & Fujii propose as future work. If the over-shooting gap shrinks for elite finishers and persists for poor ones, objection 1 is answered and the finding survives in sharpened form.
3. **Test the execution confound.** Compare the gap computed with intent-only against outcome-inclusive features — the [[intent-vs-outcome-valuation|I-VAEP/O-VAEP]] split applied to decisions rather than actions. If the gap collapses when execution difficulty enters, objection 2 explains it.
4. **Check the defender's side.** Yeung & Fujii's payoff table already contains it: is the *defender* playing the equilibrium? If defenders also deviate systematically, the game is not being played at equilibrium by either side and the prescription for shooters is conditional on fixing that.
5. **Reconcile the two granularities.** Compute both gaps on one dataset. Do players who target badly also select badly? If the two gaps are uncorrelated they are separate phenomena that happen to share a name.

## What Would Change Depending on the Answer

**If the gap predicts nothing** — then this is the clearest case in the vault of a metric measuring its own consistency, and the prescriptive claims should be withdrawn. It would also be a useful negative result: the *only* prescriptive finding this literature has produced would turn out to be circular.

**If the gap predicts outcomes** — football has a measurable, systematic coaching inefficiency, and the literature's first genuinely actionable output. That is a much larger claim than any valuation result, and it would justify the whole prescriptive turn.

**If it predicts outcomes only after conditioning on skill** — the most likely outcome, and the most useful: it would mean the finding is real but the average-player framing systematically mislabels good players as bad decision-makers, which is a correctable flaw rather than a fatal one.

## Why Nobody Has Done It

Because both papers treat the gap as an *illustration* rather than a result. Fernández et al. present it inside a control-room walkthrough demonstrating what the surface can show; Yeung & Fujii report it as a property of the equilibrium they computed. Neither frames it as a claim requiring validation, so neither validates it.

The convergence is only visible from outside — from holding both papers and noticing they say something similar. That is the kind of finding a synthesis can produce and a single paper cannot, and it is also why nobody has tested it: **no one author owns the claim.**

## See Also

- [[policy-modelling]] · [[game-theory]] · [[xsot]] · [[probability-surface]]
- [[intent-vs-outcome-valuation]] · [[predictive-validity]] · [[counterfactual-baseline]] · [[reinforcement-learning]]
- [[expected-value-possession-framework|Fernández et al. Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
- [[action-valuation-frameworks-compared]]
