---
title: "Do players decide suboptimally, or do the models only think so?"
type: question
tags: [policy-modelling, game-theory, action-valuation, evaluation, predictive-validity, counterfactual, sports-analytics, reinforcement-learning, action-space, auxiliary-loss, needs-review]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.65
provenance:
  extracted: 33%
  inferred: 57%
  generated: 9%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Do players decide suboptimally, or do the models only think so?

**Status:** Open, and **more open again** after a third framework was added.

Three frameworks with little methodologically in common each address the gap between what players do and what their model says is best. The first two report a large gap. The third **makes the size of the gap a hyperparameter**, which is a different kind of problem entirely.

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value [[probability-surface\|surface]] | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | [[game-theory\|Game-theoretic]] payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |
| **[[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]]** | **Learned $Q$ over 14 actions** | **Tunable via $\lambda_1$** | **Tunable via $\lambda_1$** | **Not reported** |

## First: They Are Not the Same Claim

Earlier vault entries described the first two as "both locating the divergence in the same place." That was too strong, and the distinction matters.

**Fernández et al. measure suboptimal *targeting within* an action.** Given that a pass is made, it often does not go to the highest-value destination on the surface. The action type is not in question.

**Yeung & Fujii measure suboptimal *selection between* actions.** Given a shot-taking situation, the shooter shoots when the equilibrium says pass. The destination is not in question.

**Nakahara et al. could measure either**, since $Q$ covers movement directions, pass and shot for every player at every timestep — and measure **neither**, reporting no observed-versus-optimal comparison at all.

So "players shoot too much" is Yeung & Fujii's claim alone. What the first two jointly support is weaker and still substantial: **a large observed-versus-optimal gap appears under two unrelated methods, on different leagues, different data types, and different decision granularities.**

That convergence is worth taking seriously precisely because the methods share no machinery — but it is convergence on the *existence* of a gap, not on its cause.

## The Fourth Objection, and It Is the Worst

> **Added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]].

The three objections below concern whether a *correctly computed* gap means what it appears to. The new one concerns whether the gap is computed at all, or partly chosen.

Nakahara et al.'s [[action-supervision]] adds an [[imitation-learning|imitation]] loss to the [[temporal-difference-learning|TD]] objective, weighted by $\lambda_1$. The authors state both failure modes explicitly: too little supervision and counterfactual action values are never properly learned; too much and the model **overfits to the actions actually taken and stops distinguishing counterfactuals**.

Read that second clause as a statement about this question:

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: drawn from the source's description of the $\lambda_1$ extremes; the general claim is not stated there. Declared on [[action-supervision]]. Also on [[reinforcement-learning]] and [[free-parameters-load-bearing]]. rests-on: source:nakahara-lambda-tradeoff]

**Turn $\lambda_1$ up and players look wise. Turn it down and they look foolish.** Nakahara et al. set it to 0.01, report a two-point ablation against 0, and never plot the gap against it.

This does not touch Fernández et al. or Yeung & Fujii directly — neither regularises toward observed behaviour. But it establishes that *a whole family of methods* has a dial on this question, and that the field has not noticed. Any future RL-based decision analysis inherits it.

It also sharpens an uncomfortable general point. **Every method here needs some assumption to fill in the counterfactual arm**, since nobody observes what would have happened. Fernández et al. use a learned surface, Yeung & Fujii use defender-removal simulations, Nakahara et al. use an imitation prior. The gap's size depends on that assumption in all three cases; only the third makes the dependence explicit and numerical.

## The Objections That Have To Be Answered First

All three gaps are measured **by the model's own value function**. "Suboptimal by the model's lights" is a statement about internal consistency, not about football.

**1. Average-player models applied to specific players.** [[expected-goals|xG]], [[xsot|xSOT]] and EPV are deliberately player-agnostic. The optimal action for an *average* player is not the optimal action for an elite finisher. A striker with genuine finishing skill *should* shoot more than an average-player equilibrium prescribes, and would be scored as over-shooting for doing the right thing.

Nakahara et al. are a partial exception worth noting: they train **ten separate agents**, one per player position in the attack, so the value functions are not fully player-agnostic. But agents are per-slot rather than per-identity and the test set is one club, so this is closer to role-conditioning than player-conditioning.

**2. Execution difficulty is unmodelled.** All three value the *intent* of the best available option, not the difficulty of executing it. A cutback worth 0.112 in expectation may be substantially harder to play than the safe ball worth 0.032. This is the [[intent-vs-outcome-valuation|intent/outcome distinction]] — the "gap" may be the price of reliability rather than an error.

Nakahara et al. inherit this in an unusually clean form: their 8-direction [[action-space-design|movement discretisation]] treats every direction as equally executable from every state, when a player at full sprint cannot turn 135° at will. **The action space encodes no dynamics constraints**, so some of the "available" counterfactual actions are not available.

**3. Equilibrium assumes a rational opponent.** Yeung & Fujii's Nash solution assumes the defender is also best-responding. If defenders systematically fail to block when they should, the shooter's true best response shifts toward shooting.

The mirror problem afflicts Nakahara et al.: their agents are **[[multi-agent-reinforcement-learning|independent]]**, so a counterfactual run is evaluated with all 21 other players' trajectories held fixed. Nobody reacts. That systematically *overstates* the value of counterfactual movement, because in reality a defender would track it.

**Objection 3 now cuts both ways.** Assuming opponents respond optimally may understate observed play's quality; assuming they do not respond at all overstates the counterfactual's. The two held frameworks that model counterfactual movement sit at opposite extremes and neither is right.

## What Can Be Settled From Held Sources

Very little, which is why this is a question rather than a finding.

What *is* established: the gap is large under two methods with no shared machinery, and a third framework demonstrates that in regularised value learning its magnitude is partly a modelling choice.

What is not established: whether the gap tracks anything outside the models. No paper tests this.

## Proposed Test

The question is one of **prescriptive validity** — does deviation from the model's optimum predict worse real outcomes? — and nothing in this vault tests it.

1. **Does the gap predict anything?** Compute per-player or per-team gap over a season and correlate with goals, points, or [[predictive-validity|next-match]] outcomes. If a larger gap predicts nothing, the models are self-consistent and uninformative about behaviour. Decisive and cheapest.
2. **Sweep $\lambda_1$ and plot the gap.** New with this ingest and nearly as cheap: one retrain per point. The curve *is* `optimality-gap-is-tunable`, measured. If the gap is flat across two orders of magnitude of $\lambda_1$, the objection is answered and the finding strengthens considerably; if it moves monotonically, any single reported gap is uninterpretable without the weight beside it. **See [[free-parameters-load-bearing]].**
3. **Condition on player skill.** Re-run with shooter-specific finishing ability, which Yeung & Fujii propose as future work. If the over-shooting gap shrinks for elite finishers and persists for poor ones, objection 1 is answered.
4. **Test the execution confound.** Compare the gap computed with intent-only against outcome-inclusive features — the [[intent-vs-outcome-valuation|I-VAEP/O-VAEP]] split applied to decisions rather than actions.
5. **Check the defender's side.** Is the *defender* playing the equilibrium? And separately: recompute Nakahara-style counterfactuals with a [[trajectory-prediction|predicted]] rather than frozen defensive response, as [[c-obso|C-OBSO]] already does for attackers. If the gap collapses, the independence assumption was carrying it.
6. **Reconcile the granularities.** Compute the targeting gap and the selection gap on one dataset. Do players who target badly also select badly? If uncorrelated they are separate phenomena sharing a name.

Steps 2 and 5 are new and both are internal to existing code. Neither requires new data.

## What Would Change Depending on the Answer

**If the gap predicts nothing** — the clearest case in the vault of a metric measuring its own consistency, and the prescriptive claims should be withdrawn. A useful negative result: the *only* prescriptive finding this literature has produced would turn out to be circular.

**If the gap predicts outcomes** — football has a measurable, systematic coaching inefficiency, and the literature's first genuinely actionable output.

**If it predicts outcomes only after conditioning on skill** — the most likely outcome, and the most useful: the finding is real but the average-player framing mislabels good players as bad decision-makers, which is correctable rather than fatal.

**If the gap moves with $\lambda_1$** — then in the RL family the question is not answerable without first fixing the prior on human competence, which is *itself* the thing under investigation. That would be circular in a deeper way than the first branch, and it would mean this family of methods cannot address this question at all without an external criterion.

## Why Nobody Has Done It

Because the papers treat the gap as an *illustration* rather than a result. Fernández et al. present it inside a control-room walkthrough; Yeung & Fujii report it as a property of the equilibrium they computed; Nakahara et al. do not report it at all, despite their framework producing it more directly than either.

That last omission is striking. A model that outputs the Q-value of every action for every player at every timestep has the observed-versus-optimal gap sitting in its output tensor, one subtraction away. **They compute everything needed and report correlations with season goals instead.**

The convergence is only visible from outside — from holding these papers and noticing they say something similar. That is the kind of finding a synthesis can produce and a single paper cannot, and it is also why nobody has tested it: **no one author owns the claim.**

## See Also

- [[policy-modelling]] · [[game-theory]] · [[xsot]] · [[probability-surface]] · [[action-space-design]]
- [[action-supervision]] · [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[imitation-learning]]
- [[intent-vs-outcome-valuation]] · [[predictive-validity]] · [[construct-validity]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[trajectory-prediction]]
- [[free-parameters-load-bearing]] · [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[expected-value-possession-framework|Fernández et al. Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
