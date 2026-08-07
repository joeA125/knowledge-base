---
title: "Are the free parameters load-bearing?"
type: question
tags: [model-selection, discounting, evaluation, sports-analytics, action-valuation, reliability, predictive-validity, space-creation, reinforcement-learning, auxiliary-loss, needs-review]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 41%
  generated: 3%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Are the free parameters load-bearing?

**Status:** Open for fourteen parameters. **One has been settled** — not by a sensitivity analysis, but by being superseded. **One now has a two-point ablation** and an explicit statement of what goes wrong at each extreme.

> **Updated 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]], which adds six and introduces a **fourth kind**. The count has gone four → five → eight → fourteen across four ingests.

| Parameter | Framework | Role | Justification given | Status |
|---|---|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | Credit decay | "Stylistic preference" | **Open** |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Hard credit cutoff | Mean possession duration | **Open** |
| $k = 5$ events | [[vdep]] | Lookahead window | Domain intuition | **Open** — inherited by [[gvdep]] |
| $4$ s | [[c-obso]] | Prediction horizon | Accuracy trade-off | **Open** |
| $w = 3$ s | [[space-occupation-gain\|SOG/SGG]] | Gain window | Expert analysts | **Open** |
| $\delta = 5$ m | [[space-occupation-gain\|SGG]] | Closeness threshold | Mean marking distance | **Open** |
| $\alpha = 3$ m | [[space-occupation-gain\|SGG]] | Minimum attraction | Avoids spurious drags | **Open** |
| $\epsilon$ (gain) | [[space-occupation-gain\|SOG/SGG]] | Gain threshold | Excludes ordinary drift | **Open** |
| **$\lambda_1 = 0.01$** | **[[action-supervision\|Nakahara et al.]]** | **Imitation weight** | **Asserted; two-point ablation** | **Open — but characterised** |
| $\lambda_2 = 0.1$ | Nakahara et al. | $L_1$ weight | Asserted | **Open** |
| $\gamma = 1$ | Nakahara et al. | Discount | "Simplicity", citing Liu et al. | **Open — but probably inert** |
| $T_{max} = 300$ frames | Nakahara et al. | Episode cap | Asserted | **Open** |
| 0.1 m/s | Nakahara et al. | Stop threshold → "idle" label | Asserted | **Open** |
| 24 km/h | Nakahara et al. | Sprint threshold → sprint labels | Asserted | **Open** |
| ~~$C \approx 3.9$~~ | ~~[[vdep]]~~ | ~~Recovery/attack weighting~~ | ~~Event frequency ratio~~ | **Superseded** |

## Four Kinds, Now Five

The count is less informative than the taxonomy. Lumping them together obscures that only some are suspect.

**Horizon parameters** ($\epsilon = 15$s, $k$, 4 s, $w$, $T_{max}$) bound how far a model looks. Likely **self-limiting**: most credit falls near the event regardless, so extending or shortening the window changes little at the margin.

**Shape parameters** ($\gamma$) govern *how* weight decays rather than *where* it stops. The most suspect: across Shelopugin's own proposed range, $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$ — nearly **two orders of magnitude** in the weight on a thirty-second-old action, offered as a stylistic choice for attacking philosophy.

**Geometric thresholds** ($\delta$, $\alpha$) define when a spatial relationship counts. They do not weight anything, they **gate whether an event is recorded at all**.

**Detection thresholds** ($\epsilon$ gain, 0.1 m/s, 24 km/h) separate signal from drift, or one label from another. Also gating rather than weighting.

**Prior-strength parameters** ($\lambda_1$) — **new with this ingest.** See below.

## The Fifth Kind Is the Most Interesting Yet

$\lambda_1$ weights an [[action-supervision|auxiliary imitation loss]] against a [[temporal-difference-learning|TD]] loss. It does not bound a horizon, shape a decay, or gate an event. It sets **how strongly the model's notion of a good action is pulled toward what humans actually did.**

| $\lambda_1 \to 0$ | $\lambda_1 \to \infty$ |
|---|---|
| Pure value learning | Pure [[imitation-learning\|imitation]] |
| Counterfactual actions unconstrained | No counterfactuals — model reproduces observed choices |
| Players look arbitrary | **Players look optimal by construction** |

That last row is why this parameter deserves separating from the others.

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: drawn from the source's own description of the $\lambda_1$ extremes; the general claim is not stated there. Declared on [[action-supervision]]. Also on [[reinforcement-learning]] and [[observed-versus-optimal-decisions]]. rests-on: source:nakahara-lambda-tradeoff]

Every other parameter here, if wrong, produces wrong *values* or the wrong *event set*. A wrong $\lambda_1$ produces a wrong **conclusion about football** — specifically, about whether players decide well. See [[observed-versus-optimal-decisions]].

## The Closest Thing to a Sweep, and Why It Isn't One

Nakahara et al. do more than anyone else here, and still not enough. They:

- Compare $\lambda_1 = 0$ against $\lambda_1 = 0.01$, reporting both losses and mean on/off-ball Q-values.
- **State the failure mode at each extreme in words** — too little supervision gives insufficient learning of counterfactual values, too much overfits to actual actions.
- **Decline to say which model is better**, on the grounds that no ground truth for a Q-value exists.

So the authors know the parameter is load-bearing, describe its behaviour qualitatively, and report two points. What is missing is the curve — and, more damagingly, any *criterion* on which a curve could be read, since they have argued there is none.

**That is a different and harder problem than the rest of this page poses.** Elsewhere the objection is that nobody swept; here it is that the authors sweep and have no metric to sweep against. See [[construct-validity]].

## The Gates Are Still the Structural Problem

A horizon parameter that is slightly wrong produces slightly wrong values. **A gate that is slightly wrong produces the wrong set of events**, and everything downstream is computed on that set.

Nakahara et al. add two of the cleanest gates in the vault. The 24 km/h sprint threshold and 0.1 m/s stop threshold do not scale any value — they **decide which of 14 action labels a frame receives**. A player at 23.8 km/h is running; at 24.1 km/h he is sprinting, and the two get different Q-values from different columns of the output layer.

Worse, these gates apply to the *action* rather than to the *outcome*, so a mislabelled frame corrupts both the state-action pair used in the TD update and the label used in the supervision loss. See [[action-space-design]].

## What GVDEP Settled, and How

[[gvdep|GVDEP]] does not sweep $C$. It **removes it**, replacing the frequency-derived constant with weights taken from [[vaep|VAEP]] at the moments ball gains and effective attacks occur — moving both terms from a **frequency scale** to a **score scale**.

The objection this page raised against $C$ — that a frequency ratio encodes how *often* each event happens and says nothing about how much each *matters* — is answered directly. **The vault's only instance of an asserted parameter fixed by principled derivation.**

Two caveats keep it from being clean: the new weights depend on classifiers whose F1 on that data is 0.08–0.15, and GVDEP inherits $k = 5$ untouched.

## The Discount Factor Now Has Three Values and No Argument

| Framework | $\gamma$ | Justification | Effect at 30 s |
|---|---|---|---|
| [[temporal-discounting\|Shelopugin]] | 0.95 /s | Stylistic preference | 21% weight retained |
| Nakahara et al. | **1** | "Simplicity", citing Liu et al. | **100% — credit spread flat** |
| Most others | n/a | Fixed horizon instead | — |

Nakahara et al.'s choice is defensible on its own terms: reward arrives only at the terminal frame of an episode capped at 300 frames, so the undiscounted return is a finite single term and convergence is not at issue. **Probably inert**, and marked so above.

But the juxtaposition is the point. Two football valuation frameworks use the same symbol at 0.95 and 1, one calling it a stylistic choice about attacking philosophy and the other a simplification, **and neither acknowledges the other's position exists.** See [[reinforcement-learning]].

## Sensitivity Analysis Is Rare, Not Absent

[[gvdep|GVDEP]] sweeps **$n\_nearest$ from 0 to 11**, reporting F1 for all four classifiers at each value — a genuine sensitivity analysis, on *which inputs to include*. [[drso|DRSO]] verifies **five velocity assumptions** against RMSE. [[physics-based-pass-probabilities|Spearman et al.]] fit by MLE grid search with contour intervals. Nakahara et al. run a two-point ablation on $\lambda_1$.

So the practice exists in this literature and is well understood. **It has still never been applied to a horizon, shape or gate parameter**, and the one prior-strength parameter got two points rather than a curve.

## The Test

Rank correlation under a parameter sweep. For each, recompute the player or team rankings across a defensible range and report Spearman's $\rho$ between the extremes.

| Result | Reading |
|---|---|
| $\rho > 0.95$ | Not load-bearing. The choice is free and the debate moot |
| $\rho \approx 0.7$–$0.9$ | Rankings shift at the margins — enough to change a shortlist, not a conclusion |
| $\rho < 0.7$ | Every published ranking is one arbitrary choice away from a different answer |

**Rank correlation rather than value correlation**, because the decisions these metrics inform are ordinal: who to sign, who to review, which moment to watch. This matters doubly for the gates, where value correlation would be measuring sums over different event sets.

**For $\gamma$, test Shelopugin's own claim.** He asserts 0.9 suits vertical attacking and 0.99 possession play. Do direct-style teams rank higher under 0.9?

**For $\delta$, check the event count first.** How many dragging incidents does SGG detect at 4, 5 and 6 m? If the count moves sharply, the gate is load-bearing before any ranking is computed.

**For $\lambda_1$, the test is different and better.** Sweep it and report, at each value, the **gap between the Q-value of the observed action and the maximum Q-value**. That curve *is* `optimality-gap-is-tunable`, measured. It needs no ground truth — it reports how much of the apparent suboptimality is the regulariser's doing. **This is the single most informative sweep available anywhere on this page**, and it is cheap: one model retrain per point.

**For the 24 km/h sprint gate**, count label flips. What fraction of frames change action label between 22 and 26 km/h? If it is small, the gate is inert; if large, every Q-value in the paper is conditional on it.

## Two Routes Nobody Uses

**Fit against a criterion.** Choose the value maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]]. The objection — that optimising for reliability might favour a degenerate metric stable because it measures little — is why predictive validity is the better of the two and both should be reported.

This route is **blocked for Nakahara et al. as they frame things**, since they argue no ground truth exists. But it is not really blocked: [[obso|OBSO]] demonstrates that next-match goals serve as an external criterion for an off-ball metric. The criterion exists; the paper does not reach for it.

**Derive it from an existing model**, as GVDEP does. Available where a parameter expresses a *trade-off between quantities another model already values*. It does not help the horizons or gates.

## What Would Change

**Horizons inert, $\gamma$, the gates and $\lambda_1$ not** — the literature's blanket omission is half-excusable, but the parameters that matter have never been examined.

**All fourteen inert** — a useful negative result, and the vault's repeated complaint should be softened.

**$\lambda_1$ load-bearing** — this is the consequential branch. It would mean the observed-versus-optimal finding, which [[observed-versus-optimal-decisions]] treats as possibly the literature's only actionable output, is partly an artefact of a regularisation weight in at least one framework.

## Why Nobody Has Done It

A sensitivity analysis can only weaken a paper: it shows results are robust, which reviewers assume anyway, or shows they are not, which invites the question of why *this* value was chosen.

GVDEP is a partial counter-example — it published a sweep and the sweep was **flattering**, showing its method works with fewer inputs than its predecessor assumed. That suggests the asymmetry breaks when the sweep is over something the authors *want* to show is unnecessary.

Nakahara et al. are a second, subtler counter-example: they publish the ablation that shows their contribution *helps* ($\lambda_1 = 0$ against 0.01) and stop exactly where a sweep would start asking whether it helps too much.

## See Also

- [[model-selection]] · [[gvdep]] · [[vdep]] · [[temporal-discounting]] · [[space-occupation-gain]] · [[action-supervision]]
- [[split-half-reliability]] · [[predictive-validity]] · [[construct-validity]] · [[pass-carry-reward]] · [[c-obso]] · [[drso]] · [[obso]]
- [[action-valuation]] · [[action-space-design]] · [[reinforcement-learning]] · [[temporal-difference-learning]] · [[imitation-learning]]
- [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Shelopugin]] · [[football-defence-evaluation-vdep|VDEP]] · [[generalized-vdep-euro-location-analysis|GVDEP]] · [[wide-open-spaces-space-creation|Wide Open Spaces]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]]
