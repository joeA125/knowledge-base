---
title: "Are the five free parameters load-bearing?"
type: question
tags: [model-selection, discounting, evaluation, sports-analytics, action-valuation, reliability, predictive-validity, needs-review]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Are the five free parameters load-bearing?

**Status:** Open. Five frameworks, five asserted parameters, **zero sensitivity analyses between them.**

| Parameter | Framework | Role | Stated justification |
|---|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | Credit decay | "Stylistic preference" |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Hard credit cutoff | Mean possession duration |
| $k = 5$ events | [[vdep]] | Lookahead window | Domain intuition |
| $C \approx 3.9$ | [[vdep]] | Recovery/attack weighting | Event frequency ratio |
| $4$ s | [[c-obso]] | Prediction horizon | Accuracy trade-off |

[[obso|Spearman]] is the exception and shows it is possible: all six of his parameters are MAP-fitted with stated priors, because they carry **physical units** and can inherit priors from prior measurement. See [[model-selection]].

## They Are Not All the Same Kind of Parameter

Lumping them together, as the vault has, obscures that only some are suspect.

**Horizon parameters** ($\epsilon$, $k$, 4 s) bound how far a model looks. These are likely **self-limiting**: most credit falls near the event regardless, so extending or shortening the window changes little at the margin. The 4 s horizon is additionally the best-justified of the five, chosen against measured prediction error.

**$\gamma$ is a shape parameter**, not a horizon, and it is the most suspect. Across the range its own author proposes, $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$ — nearly **two orders of magnitude** in the weight given to a thirty-second-old action. Shelopugin offers it explicitly as a stylistic choice for attacking philosophy, which is a claim that rankings *should* change with it. He never shows by how much.

**$C$ is a trade-off weight**, and different in kind again. It sets the exchange rate between two proxies — how many prevented attacks one ball recovery is worth. It comes from the **frequency ratio** of the two events, which encodes how often each happens and says nothing about how much each matters. A team that recovers often and concedes territory often will look better or worse depending entirely on $C$, and its value was never argued for.

So the prior should be: $\gamma$ and $C$ matter, the horizons probably do not.

## The Test

Rank correlation under a parameter sweep. For each parameter, recompute the player or team rankings across a defensible range and report Spearman's $\rho$ between the extremes.

| Result | Reading |
|---|---|
| $\rho > 0.95$ | The parameter is not load-bearing. The choice is free and the debate moot |
| $\rho \approx 0.7$–$0.9$ | Rankings shift at the margins — enough to change a shortlist, not a conclusion |
| $\rho < 0.7$ | Every published ranking is one arbitrary choice away from a different answer |

Reporting **rank correlation rather than value correlation** is the right summary, because the decisions these metrics inform are ordinal: who to sign, who to review, which moment to watch. Values can move a lot while rankings hold.

Two refinements worth adding:

**For $\gamma$, test the author's own claim.** He asserts 0.9 suits vertical attacking and 0.99 suits possession play. That is testable: do teams with direct styles rank higher under 0.9? If the parameter behaves as claimed, it is a *feature* and should be reported as a family of metrics. If rankings barely move, the stylistic framing is unfounded.

**For $C$, sweep beyond the frequency ratio.** The interesting range is not around 3.9 but across plausible *importance* ratios, which could be anywhere from 1 to 10. If VDEP's team ordering is stable across that, the arbitrary choice does not matter; if not, VDEP reports one of many possible orderings.

## The Better Alternative Nobody Uses

All five could be **fitted rather than asserted**, by a criterion the vault already has: choose the value maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]].

That reframes a modelling preference as what it is — a [[model-selection]] problem — and it is cheap, since both criteria are already computed for other purposes. It also produces a defensible answer where "stylistic preference" produces an unfalsifiable one.

The objection is that optimising for reliability might favour a degenerate metric that is stable because it measures little. That is real and is why [[predictive-validity]] is the better criterion of the two, and why the two should be reported together.

## What Would Change

**If the horizons are inert and $\gamma$, $C$ are not** — the literature's blanket omission is half-excusable, but the two parameters that matter have never been examined, and that should be stated plainly on [[pass-carry-reward|PCR]] and [[vdep]].

**If all five are inert** — a genuinely useful negative result. It would mean the vault's repeated complaint about unjustified parameters, which now appears on [[model-selection]], [[action-valuation]] and the [[action-valuation-frameworks-compared|synthesis]], is overstated and should be softened.

**If rankings move substantially** — then no framework here reports *a* ranking; each reports one point in a family, and comparisons between frameworks are confounded by parameter choice on top of everything else.

## Why Nobody Has Done It

A sensitivity analysis can only weaken a paper. It either shows the results are robust — which reviewers assume anyway — or shows they are not, which invites the question of why *this* value was chosen. There is no version of the outcome that helps the authors.

That asymmetry is why it is a good task for a synthesis rather than an author: **the vault has no stake in any of the five values being right.**

## See Also

- [[model-selection]] · [[temporal-discounting]] · [[split-half-reliability]] · [[predictive-validity]]
- [[pass-carry-reward]] · [[vdep]] · [[c-obso]] · [[expected-possession-value]] · [[obso]]
- [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Shelopugin Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
