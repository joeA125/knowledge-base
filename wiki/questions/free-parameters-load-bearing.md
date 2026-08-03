---
title: "Are the free parameters load-bearing?"
type: question
tags: [model-selection, discounting, evaluation, sports-analytics, action-valuation, reliability, predictive-validity, needs-review]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 45%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Are the free parameters load-bearing?

**Status:** Open for four parameters. **One has been settled** — not by a sensitivity analysis, but by being superseded.

> **Updated, 2026-07-27** on ingest of [[generalized-vdep-euro-location-analysis|GVDEP]]. This page previously listed five asserted parameters and claimed zero sensitivity analyses across the literature. Both figures have changed.

| Parameter | Framework | Role | Justification given | Status |
|---|---|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | Credit decay | "Stylistic preference" | **Open** |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Hard credit cutoff | Mean possession duration | **Open** |
| $k = 5$ events | [[vdep]] | Lookahead window | Domain intuition | **Open** — inherited unchanged by [[gvdep]] |
| $4$ s | [[c-obso]] | Prediction horizon | Accuracy trade-off | **Open** |
| ~~$C \approx 3.9$~~ | ~~[[vdep]]~~ | ~~Recovery/attack weighting~~ | ~~Event frequency ratio~~ | **Superseded** |

## What GVDEP Settled, and How

[[gvdep|GVDEP]] does not sweep $C$ and report sensitivity. It **removes it**, replacing the frequency-derived constant with weights taken from [[vaep|VAEP]] evaluated at the moments ball gains and effective attacks occur:

$$w_{gains} = \frac{1}{|Ev_{gains}|}\sum_{j \in Ev_{gains}} \text{sign}(Team_j)\,V_{vaep}(s_j)$$

Both terms move from a **frequency scale** to a **score scale**. The objection this page raised against $C$ — that a frequency ratio encodes how *often* each event happens and says nothing about how much each *matters* — is answered directly.

**This is the vault's only instance of an asserted parameter being fixed by principled derivation** rather than left unexamined or swept.

Two caveats keep it from being a clean win. The new weights depend on VAEP's $P_{scores}$ and $P_{concedes}$ classifiers, whose F1 on this data is **0.10–0.13 and 0.08–0.15** — a principled weight computed from an unreliable model. And GVDEP inherits $k = 5$ from VDEP without re-examination, so one parameter was fixed while another passed through untouched.

## Sensitivity Analysis Is Rare, Not Absent

The previous framing — "zero sensitivity analyses" — was wrong. GVDEP sweeps **$n\_nearest$ from 0 to 11**, reporting F1 for all four classifiers at each value. That is a genuine sensitivity analysis, on *which inputs to include* rather than on a horizon or weighting parameter.

Its result is substantive: ball-gain prediction saturates at three or four players; scores, concedes and being-attacked gain nothing from player positions at all. And concedes prediction gets **worse** as players are added, 0.15 falling to 0.08.

**This removes the excuse.** A group that sweeps one parameter and publishes the curve could sweep another. The absence of horizon-parameter sensitivity analysis is a choice, not a methodological blind spot in the field.

## The Four Remaining Are Not Alike

**Horizon parameters** ($\epsilon$, $k$, 4 s) bound how far a model looks. Likely **self-limiting**: most credit falls near the event regardless. The 4 s horizon is the best-justified of the four, chosen against measured prediction error.

**$\gamma$ is a shape parameter**, and the most suspect. Across its author's own proposed range, $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$ — nearly **two orders of magnitude** in the weight on a thirty-second-old action. Shelopugin offers it as a stylistic choice for attacking philosophy, which is a claim that rankings *should* change with it. He never shows by how much.

With $C$ gone, **the trade-off-weight category is now empty**, and what remains is one shape parameter and three horizons. That is a narrower and more tractable question than the original five.

## The Test

Rank correlation under a parameter sweep. For each, recompute the player or team rankings across a defensible range and report Spearman's $\rho$ between the extremes.

| Result | Reading |
|---|---|
| $\rho > 0.95$ | Not load-bearing. The choice is free and the debate moot |
| $\rho \approx 0.7$–$0.9$ | Rankings shift at the margins — enough to change a shortlist, not a conclusion |
| $\rho < 0.7$ | Every published ranking is one arbitrary choice away from a different answer |

**Rank correlation rather than value correlation** is the right summary, because the decisions these metrics inform are ordinal: who to sign, who to review, which moment to watch.

**For $\gamma$, test the author's own claim.** He asserts 0.9 suits vertical attacking and 0.99 suits possession play. Do teams with direct styles rank higher under 0.9? If the parameter behaves as claimed, it is a *feature* and should be reported as a family of metrics. If rankings barely move, the stylistic framing is unfounded.

## Two Routes Nobody Uses

**Fit against a criterion.** Choose the value maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]]. Both are already computed for other purposes. The objection — that optimising for reliability might favour a degenerate metric stable because it measures little — is real, which is why predictive validity is the better of the two and both should be reported.

**Derive it from an existing model**, as GVDEP does. This route was invisible before that paper and is now demonstrated: where a parameter expresses a *trade-off between quantities that another model already values*, the other model can supply the exchange rate.

Note the second route does not help the horizons. There is no existing model that values "how far back credit should propagate", so $\gamma$, $\epsilon$, $k$ and 4 s remain candidates for the first route only.

## What Would Change

**If the horizons are inert and $\gamma$ is not** — the literature's omission is half-excusable, but the one parameter that matters has never been examined, and that should be stated plainly on [[pass-carry-reward|PCR]].

**If all four are inert** — a useful negative result, and the vault's repeated complaint about unjustified parameters should be softened.

**If rankings move substantially** — no framework here reports *a* ranking; each reports one point in a family, and cross-framework comparison is confounded by parameter choice on top of everything else.

## Why Nobody Has Done It

A sensitivity analysis can only weaken a paper: it shows results are robust, which reviewers assume anyway, or shows they are not, which invites the question of why *this* value was chosen.

GVDEP is a partial counter-example — it published a sweep and the sweep was flattering, showing its method works with fewer inputs than its predecessor assumed. That suggests the asymmetry breaks when the sweep is over something the authors *want* to show is unnecessary, rather than over a value they had to choose.

## See Also

- [[model-selection]] · [[gvdep]] · [[vdep]] · [[temporal-discounting]]
- [[split-half-reliability]] · [[predictive-validity]] · [[pass-carry-reward]] · [[c-obso]] · [[obso]]
- [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Shelopugin Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]]
