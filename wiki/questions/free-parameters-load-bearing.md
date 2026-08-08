---
title: "Are the free parameters load-bearing?"
type: question
tags: [model-selection, discounting, evaluation, sports-analytics, action-valuation, reliability, predictive-validity, space-creation, reinforcement-learning, auxiliary-loss, domain-adaptation, training-technique, needs-review]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 56%
  inferred: 39%
  generated: 4%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Are the free parameters load-bearing?

**Status:** Open for sixteen parameters. **One has been settled** — by being superseded, not by a sensitivity analysis. **One has a two-point ablation.** And one, newly, is a parameter nobody in this literature reports at all.

> **Updated 2026-08-07** on ingest of [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] — the paper this page named as the place the $\lambda_1$ answer would be found. **It is not there.** See "The Acquisition That Did Not Help".
>
> The count has gone four → five → eight → fourteen → sixteen across five ingests.

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
| **$\lambda_1$, $\lambda_2$** | **[[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]]** | Imitation and $L_2$ weights | **None — values not reported** | **Open, and unreported** |
| **Training steps (0.5M)** | **Fujii et al.** | **Where on the imitation/reward frontier the model lands** | None — treated as hygiene | **Open — new sixth kind** |
| ~~$C \approx 3.9$~~ | ~~[[vdep]]~~ | ~~Recovery/attack weighting~~ | ~~Event frequency ratio~~ | **Superseded** |

## Five Kinds, Now Six

The count is less informative than the taxonomy. Lumping them together obscures that only some are suspect.

**Horizon parameters** ($\epsilon = 15$s, $k$, 4 s, $w$, $T_{max}$) bound how far a model looks. Likely **self-limiting**: most credit falls near the event regardless.

**Shape parameters** ($\gamma$) govern *how* weight decays rather than *where* it stops. Across Shelopugin's own proposed range, $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$ — nearly **two orders of magnitude** in the weight on a thirty-second-old action, offered as a stylistic choice.

**Geometric thresholds** ($\delta$, $\alpha$) define when a spatial relationship counts. They **gate whether an event is recorded at all**.

**Detection thresholds** ($\epsilon$ gain, 0.1 m/s, 24 km/h) separate signal from drift, or one label from another. Also gating.

**Prior-strength parameters** ($\lambda_1$) set how hard a value function is pulled toward observed behaviour.

**Stopping parameters** (training steps) — **new 2026-08-07.** See below.

## The Sixth Kind: Where You Stop Is a Modelling Choice

[[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] report that on the chase-and-escape task,

> with increased training steps, the DQAAS first learned the ability to maximize a reward and then learned the reproducibility at the expense of the reward

That relocates the dial this page has been chasing. The vault had assumed the imitation/reward balance was governed by a **loss weight**. It is also governed by **how long you train**.

> ### `where-you-stop-is-a-modelling-choice`
> **When imitation and reward objectives trade off during training rather than only across hyperparameter settings, the training-step count is a free parameter with the same status as a loss weight — and it is almost never reported as one.**
> ^[generated: no source frames early stopping this way; drawn from the observed reward-then-reproducibility ordering. Declared on [[imitation-reward-tradeoff]]. rests-on: source:fujii-training-step-ordering]

This is uncomfortable because early stopping is normally treated as hygiene rather than as a substantive claim. Here, **stopping early yields a reward-maximising agent and stopping late an imitative one — two different scientific claims about football, selected by a compute budget.**

Note the ordering is the reverse of the naive expectation. A model pre-trained on demonstrations might be expected to start imitative and drift toward reward; the opposite happened.

## The Acquisition That Did Not Help

> ⚠️ **The vault predicted where an answer would be and was wrong. Recorded because the failure is informative about how to read this literature.**

Three pages, including this one, stated that arXiv:2305.13030 "would presumably settle" the $\lambda_1$ question. It has been acquired. What it contains:

- **No value for $\lambda_1$ or $\lambda_2$**, anywhere. The text says only that "the $\lambda$ parameters control the weight of these losses."
- An ablation over **which losses are present** (DQN / DQfD / DQfAD / DQAS / DQAAS), not over their weights.
- No sweep and no curve.

**The dedicated methodological paper proposing action supervision is less specific about its weight than the applied paper that borrows it.** Nakahara et al. at least state $\lambda_1 = 0.01$ and compare against $0$.

The lesson generalises past this instance: **a paper's contribution being a mechanism does not mean it characterises that mechanism's parameters.** Methods papers optimise for showing a method works, which is an ablation question ("is this component needed?"), not a calibration question ("how much of it?"). Expecting the latter from the former was the vault's error.

The $\lambda_1$ question is unchanged and harder, because the obvious place has now been checked.

## The Fifth Kind Remains the Most Consequential

$\lambda_1$ does not bound a horizon, shape a decay, or gate an event. It sets **how strongly the model's notion of a good action is pulled toward what humans actually did.**

| $\lambda_1 \to 0$ | $\lambda_1 \to \infty$ |
|---|---|
| Pure value learning | Pure [[imitation-learning\|imitation]] |
| Counterfactual actions unconstrained | No counterfactuals — model reproduces observed choices |
| Players look arbitrary | **Players look optimal by construction** |

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: **strengthened 2026-08-07.** Previously rested on Nakahara et al.'s verbal description alone; Fujii et al. now measure the reward/imitation trade-off directly, though against training steps and method choice rather than $\lambda_1$. The dial is real; $\lambda_1$ is not the only knob. Declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

Every other parameter here, if wrong, produces wrong *values* or the wrong *event set*. A wrong $\lambda_1$ produces a wrong **conclusion about football**. See [[observed-versus-optimal-decisions]] and [[imitation-reward-tradeoff]].

## The Gates Are Still the Structural Problem

A horizon parameter that is slightly wrong produces slightly wrong values. **A gate that is slightly wrong produces the wrong set of events.**

Nakahara et al.'s 24 km/h sprint threshold and 0.1 m/s stop threshold do not scale any value — they **decide which of 14 action labels a frame receives**. A player at 23.8 km/h is running; at 24.1 km/h he is sprinting, and the two draw Q-values from different columns of the output layer.

These gates apply to the *action* rather than the *outcome*, so a mislabelled frame corrupts both the state-action pair in the TD update and the label in the supervision loss. See [[action-space-design]].

## What GVDEP Settled, and How

[[gvdep|GVDEP]] does not sweep $C$. It **removes it**, replacing the frequency-derived constant with weights taken from [[vaep|VAEP]] at the moments ball gains and effective attacks occur — moving both terms from a **frequency scale** to a **score scale**.

**The vault's only instance of an asserted parameter fixed by principled derivation.** Two caveats: the new weights depend on classifiers whose F1 is 0.08–0.15, and GVDEP inherits $k = 5$ untouched.

## The Discount Factor Has Three Values and No Argument

| Framework | $\gamma$ | Justification | Effect at 30 s |
|---|---|---|---|
| [[temporal-discounting\|Shelopugin]] | 0.95 /s | Stylistic preference | 21% weight retained |
| Nakahara et al. | **1** | "Simplicity", citing Liu et al. | **100% — credit spread flat** |
| Fujii et al. | Stated as $\gamma \in (0,1]$ | **Value not reported** | Unknown |
| Most others | n/a | Fixed horizon instead | — |

Nakahara et al.'s choice is defensible: reward arrives only at the terminal frame of a bounded episode, so the undiscounted return is a finite single term. **Probably inert.**

The juxtaposition is the point. Two football valuation frameworks use the same symbol at 0.95 and 1, and a third from the same group as the second does not report it. See [[reinforcement-learning]].

## Sensitivity Analysis Is Rare, Not Absent

[[gvdep|GVDEP]] sweeps **$n\_nearest$ from 0 to 11**, reporting F1 for all four classifiers at each value. [[drso|DRSO]] verifies **five velocity assumptions** against RMSE. [[physics-based-pass-probabilities|Spearman et al.]] fit by MLE grid search with contour intervals. Nakahara et al. run a two-point ablation on $\lambda_1$. Fujii et al. ablate loss *presence* across five model variants and report five random seeds.

So the practice exists and is well understood. **It has still never been applied to a horizon, shape or gate parameter**, and the two prior-strength parameters got two points and zero points respectively.

## The Test

Rank correlation under a parameter sweep. For each, recompute the player or team rankings across a defensible range and report Spearman's $\rho$ between the extremes.

| Result | Reading |
|---|---|
| $\rho > 0.95$ | Not load-bearing. The choice is free and the debate moot |
| $\rho \approx 0.7$–$0.9$ | Rankings shift at the margins — enough to change a shortlist, not a conclusion |
| $\rho < 0.7$ | Every published ranking is one arbitrary choice away from a different answer |

**Rank correlation rather than value correlation**, because the decisions these metrics inform are ordinal.

**For $\gamma$, test Shelopugin's own claim.** Do direct-style teams rank higher under 0.9?

**For $\delta$, check the event count first.** If the count of detected dragging incidents moves sharply between 4 and 6 m, the gate is load-bearing before any ranking is computed.

**For $\lambda_1$, the test is unchanged and still unrun.** Sweep it and report, at each value, the **gap between the Q-value of the observed action and the maximum Q-value**. That curve *is* `optimality-gap-is-tunable`, measured. It needs no ground truth. **Still the single most informative sweep available on this page**, and cheap — one retrain per point.

**For training steps, the test is nearly free.** Checkpoint across training and plot reward against [[dynamic-time-warping|DTW]] distance. Fujii et al. have the checkpoints; they report two columns (pre-trained, 0.5M) where a curve would cost nothing extra.

**For the 24 km/h sprint gate**, count label flips between 22 and 26 km/h.

## Two Routes Nobody Uses

**Fit against a criterion.** Choose the value maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]]. Nakahara et al. treat this as blocked because no ground truth exists — but [[obso|OBSO]] demonstrates next-match goals serve as an external criterion for an off-ball metric.

**Derive it from an existing model**, as GVDEP does. Available where a parameter expresses a *trade-off between quantities another model already values*.

## What Would Change

**Horizons inert, $\gamma$, the gates and $\lambda_1$ not** — the literature's blanket omission is half-excusable, but the parameters that matter have never been examined.

**All sixteen inert** — a useful negative result; the vault's repeated complaint should soften.

**$\lambda_1$ or the stopping point load-bearing** — the consequential branch. The observed-versus-optimal finding, which [[observed-versus-optimal-decisions]] treats as possibly the literature's only actionable output, would be partly an artefact of a weight or a compute budget.

## Why Nobody Has Done It

A sensitivity analysis can only weaken a paper: it shows results are robust, which reviewers assume anyway, or shows they are not, which invites the question of why *this* value was chosen.

GVDEP is a partial counter-example — it published a sweep and the sweep was **flattering**, showing its method works with fewer inputs than its predecessor assumed. The asymmetry breaks when the sweep is over something the authors *want* to show is unnecessary.

Nakahara et al. and Fujii et al. are subtler counter-examples. Both publish the ablation showing their contribution *helps*, and both stop exactly where a sweep would begin asking whether it helps too much. **The ablation is the flattering experiment; the sweep is the risky one.**

## See Also

- [[model-selection]] · [[gvdep]] · [[vdep]] · [[temporal-discounting]] · [[space-occupation-gain]] · [[action-supervision]]
- [[imitation-reward-tradeoff]] · [[domain-adaptation]] · [[dynamic-time-warping]] · [[deep-q-network]] · [[imitation-learning]]
- [[split-half-reliability]] · [[predictive-validity]] · [[construct-validity]] · [[pass-carry-reward]] · [[c-obso]] · [[drso]] · [[obso]]
- [[action-valuation]] · [[action-space-design]] · [[reinforcement-learning]] · [[temporal-difference-learning]]
- [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Shelopugin]] · [[football-defence-evaluation-vdep|VDEP]] · [[generalized-vdep-euro-location-analysis|GVDEP]] · [[wide-open-spaces-space-creation|Wide Open Spaces]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
