---
title: "Do players decide suboptimally, or do the models only think so?"
type: question
tags: [policy-modelling, game-theory, action-valuation, evaluation, predictive-validity, counterfactual, sports-analytics, reinforcement-learning, action-space, auxiliary-loss, imitation-learning, domain-adaptation, needs-review]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.7
provenance:
  extracted: 38%
  inferred: 52%
  generated: 9%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Do players decide suboptimally, or do the models only think so?

**Status:** Open. The fourth objection — that the gap is partly tunable — was added on inference and is **now supported by measurement**.

Three frameworks with little methodologically in common each address the gap between what players do and what their model says is best. The first two report a large gap. The third **makes the size of the gap a modelling choice**, and a fourth source measures the trade-off that makes it one.

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value [[probability-surface\|surface]] | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | [[game-theory\|Game-theoretic]] payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | Learned $Q$ over 14 actions | **Tunable via $\lambda_1$** | **Tunable via $\lambda_1$** | **Not reported** |
| [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] | DQAAS in a simulator | — | — | **Trade-off measured, gap not** |

## First: They Are Not the Same Claim

Earlier vault entries described the first two as "both locating the divergence in the same place." Too strong.

**Fernández et al. measure suboptimal *targeting within* an action.** Given that a pass is made, it often does not go to the highest-value destination. The action type is not in question.

**Yeung & Fujii measure suboptimal *selection between* actions.** Given a shot-taking situation, the shooter shoots when the equilibrium says pass. The destination is not in question.

**Nakahara et al. could measure either**, since $Q$ covers movement, pass and shot for every player at every timestep — and measure **neither**.

So "players shoot too much" is Yeung & Fujii's claim alone. What the first two jointly support is weaker and still substantial: **a large observed-versus-optimal gap appears under two unrelated methods, on different leagues, data types and decision granularities.**

## The Fourth Objection, Now Measured

The three objections below concern whether a *correctly computed* gap means what it appears to. The fourth concerns whether the gap is computed at all, or partly chosen.

[[action-supervision]] adds an [[imitation-learning|imitation]] loss to the [[temporal-difference-learning|TD]] objective, weighted by $\lambda_1$. Nakahara et al. state both failure modes: too little supervision and counterfactual values are never learned; too much and the model **overfits to the actions actually taken and stops distinguishing counterfactuals**.

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: **strengthened 2026-08-07.** Previously rested on Nakahara et al.'s verbal description alone. Fujii et al. now measure the reward/imitation trade-off directly, reporting that reward and DTW distance "had a trade-off relationship". The dial is real; $\lambda_1$ is not the only knob. Declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

**Turn the supervision up and players look wise. Turn it down and they look foolish.**

### And the dial has a second knob nobody reports

> **Added 2026-08-07.** The more uncomfortable half.

Fujii et al. observe that on the chase-and-escape task, the model

> first learned the ability to maximize a reward and then learned the reproducibility at the expense of the reward

So the position on the imitation/reward frontier moves with **training time**, not only with a loss weight. Stopping early gives a reward-maximising agent; stopping late gives an imitative one. See [[imitation-reward-tradeoff]] and [[free-parameters-load-bearing]], where this is recorded as a sixth kind of free parameter.

For this question that means: **even a paper that reported $\lambda_1$ and swept it would not have pinned the gap down**, because the training budget moves it too. Any reported optimality gap in this family needs both numbers beside it, and no paper reports either.

Note this does not touch Fernández et al. or Yeung & Fujii directly — neither regularises toward observed behaviour. But it establishes that a whole family of methods has dials on this question and the field has not noticed.

**The general point is sharper than it looks.** Every method here needs some assumption to fill the counterfactual arm, since nobody observes what would have happened. Fernández et al. use a learned surface, Yeung & Fujii defender-removal simulations, Nakahara et al. an imitation prior. The gap's size depends on that assumption in all three cases; only the third makes the dependence numerical.

## The Objections That Have To Be Answered First

All three gaps are measured **by the model's own value function**. "Suboptimal by the model's lights" is a statement about internal consistency, not about football.

**1. Average-player models applied to specific players.** [[expected-goals|xG]], [[xsot|xSOT]] and EPV are deliberately player-agnostic. A striker with genuine finishing skill *should* shoot more than an average-player equilibrium prescribes, and would be scored as over-shooting for doing the right thing.

Nakahara et al. are a partial exception: ten agents, one per attacking slot, so value functions are role-conditional rather than fully player-agnostic. Closer to role-conditioning than player-conditioning, on one club.

**2. Execution difficulty is unmodelled.** All three value the *intent* of the best available option, not the difficulty of executing it. A cutback worth 0.112 may be much harder to play than the safe ball worth 0.032. This is the [[intent-vs-outcome-valuation|intent/outcome distinction]] — the "gap" may be the price of reliability.

Nakahara et al. inherit this cleanly: their 8-direction [[action-space-design|movement discretisation]] treats every direction as equally executable from every state, when a player at full sprint cannot turn 135° at will. **The action space encodes no dynamics constraints**, so some "available" counterfactual actions are not available.

**3. Equilibrium assumes a rational opponent.** Yeung & Fujii's Nash solution assumes the defender is also best-responding. If defenders systematically fail to block, the shooter's true best response shifts toward shooting.

The mirror problem afflicts Nakahara et al.: their agents are [[multi-agent-reinforcement-learning|independent]], so a counterfactual run is evaluated with all 21 other players held fixed. Nobody reacts. That systematically *overstates* counterfactual movement's value.

**Objection 3 cuts both ways.** Assuming opponents respond optimally may understate observed play's quality; assuming they do not respond at all overstates the counterfactual's. The two held frameworks that model counterfactual movement sit at opposite extremes and neither is right.

> **Partial answer 2026-08-07.** Fujii et al. test whether *centralised* MARL — agents that share information — fixes anything, using CDS (Li et al., 2021), and find it does not. That is evidence the independence assumption is not the binding constraint on the forward approach, though it says nothing about whether frozen-world counterfactuals distort *valuation*. See [[multi-agent-reinforcement-learning]].

## What Can Be Settled From Held Sources

Very little, which is why this is a question.

**Established:** the gap is large under two methods with no shared machinery, and in regularised value learning its magnitude is partly a modelling choice — now with measurement behind the trade-off rather than only a verbal description.

**Not established:** whether the gap tracks anything outside the models. No paper tests this.

## Proposed Test

The question is one of **prescriptive validity** — does deviation from the model's optimum predict worse real outcomes? — and nothing in this vault tests it.

1. **Does the gap predict anything?** Compute per-player or per-team gap over a season and correlate with goals, points, or [[predictive-validity|next-match]] outcomes. If a larger gap predicts nothing, the models are self-consistent and uninformative. Decisive and cheapest.
2. **Sweep $\lambda_1$ and plot the gap.** One retrain per point. The curve *is* `optimality-gap-is-tunable`, measured. **Still unrun** — and the paper the vault expected to run it has now been checked and did not. See [[free-parameters-load-bearing]].
3. **Checkpoint across training and plot the same thing.** New with the Fujii ingest and nearly free — the checkpoints already exist. If the gap moves with training steps as the reward/reproducibility ordering suggests, any single reported gap is uninterpretable without a step count.
4. **Condition on player skill.** Re-run with shooter-specific finishing ability. If the over-shooting gap shrinks for elite finishers and persists for poor ones, objection 1 is answered.
5. **Test the execution confound.** Compare the gap computed with intent-only against outcome-inclusive features — the [[intent-vs-outcome-valuation|I-VAEP/O-VAEP]] split applied to decisions.
6. **Check the defender's side.** Recompute Nakahara-style counterfactuals with a [[trajectory-prediction|predicted]] rather than frozen defensive response, as [[c-obso|C-OBSO]] already does for attackers. If the gap collapses, the independence assumption was carrying it.
7. **Reconcile the granularities.** Compute the targeting gap and the selection gap on one dataset. If uncorrelated they are separate phenomena sharing a name.

Steps 2, 3 and 6 are internal to existing code and need no new data.

## What Would Change Depending on the Answer

**If the gap predicts nothing** — the clearest case in the vault of a metric measuring its own consistency, and the prescriptive claims should be withdrawn.

**If the gap predicts outcomes** — football has a measurable coaching inefficiency, and the literature's first genuinely actionable output.

**If it predicts outcomes only after conditioning on skill** — most likely, and most useful: the finding is real but the average-player framing mislabels good players as bad decision-makers.

**If the gap moves with $\lambda_1$ or with training steps** — then in the RL family the question is not answerable without first fixing a prior on human competence, which is *itself* the thing under investigation. Circular in a deeper way than the first branch, and it would mean this family cannot address this question without an external criterion.

## Why Nobody Has Done It

The papers treat the gap as an *illustration* rather than a result. Fernández et al. present it inside a control-room walkthrough; Yeung & Fujii report it as a property of their equilibrium; Nakahara et al. do not report it at all, despite their framework producing it more directly than either.

That last omission is striking. A model outputting the Q-value of every action for every player at every timestep has the observed-versus-optimal gap sitting in its output tensor, one subtraction away. **They compute everything needed and report correlations with season goals instead.**

The convergence is only visible from outside — from holding these papers and noticing they say something similar. That is the kind of finding a synthesis can produce and a single paper cannot, and it is also why nobody has tested it: **no one author owns the claim.**

## See Also

- [[policy-modelling]] · [[game-theory]] · [[xsot]] · [[probability-surface]] · [[action-space-design]]
- [[action-supervision]] · [[imitation-reward-tradeoff]] · [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[imitation-learning]] · [[domain-adaptation]]
- [[intent-vs-outcome-valuation]] · [[predictive-validity]] · [[construct-validity]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[trajectory-prediction]]
- [[free-parameters-load-bearing]] · [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[expected-value-possession-framework|Fernández et al.]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
