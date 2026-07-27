---
title: "VDEP (Valuing Defense by Estimating Probabilities)"
type: concept
tags: [sports-analytics, defensive-valuation, action-valuation, off-ball, proxy-target, gradient-boosting, optical-tracking-data, evaluation, single-source]
sources: [raw/papers/football_defence_evaluation.md]
confidence: 0.8
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# VDEP (Valuing Defense by Estimating Probabilities)

[[football-defence-evaluation-vdep|Toda, Teranishi, Kushiro & Fujii's (2022)]] team-level defensive metric. Named and built as a deliberate counterpart to [[vaep]], modifying its published code, and differing on exactly two things: **what is predicted**, and **what the model can see**.

## The Construction

Two classifiers over the next $k = 5$ events:

$$V_{vdep}(S_i) = P_{recoveries}(S_i) - C \cdot P_{attacked}(S_i)$$

- $P_{recoveries}$ — probability the defending team wins the ball back
- $P_{attacked}$ — probability the defending team concedes an *effective attack*
- $C \approx 3.9$, the observed frequency ratio of the two events

Team value is the mean of $V_{vdep}$ over that team's events in a match. XGBoost for both classifiers.

## The Two Departures from VAEP

**Frequent targets instead of rare ones.** VAEP predicts scoring and conceding. VDEP predicts recovery and being attacked, which occur roughly 90× and 35× more often than goals in the same dataset. See [[rare-event-proxy-targets]].

**Off-ball state.** VAEP's state is on-ball action features. VDEP's is $s_i = [a_i, o_i]$, where $o_i$ holds all 22 players' coordinates and their distances from the ball, **sorted by proximity**. That sort makes the representation permutation-invariant — "nearest defender" occupies a fixed feature slot regardless of identity — which is what lets a tree ensemble use it at all.

[[shap]] confirms the off-ball features dominate: nearest-defender distance is the top contributor to $P_{recoveries}$, and the nearest attacker's $x$-position to $P_{attacked}$.

## What "Effective Attack" Buys

The target is an effective attack — a chain ending in a shot **or** entering the penalty area — rather than a shot.

This is a better definition than it first appears. A forward who receives in the box and passes instead of shooting has still beaten the defence; scoring it as a non-event would reward defences for the attacker's choice. Defining the failure as *territorial penetration* rather than *shot taken* decouples the defensive assessment from attacking decision-making.

## Results Worth Knowing

$P_{recoveries}$ F1 = 0.522, $P_{attacked}$ F1 = 0.484, against VAEP's $P_{scores}$ 0.201 and $P_{concedes}$ **0.000**. VAEP's conceding classifier identifies no true positives at all on this data.

On correlations, $R_{vdep}$ relates similarly to match points (0.464) and season points (0.397), while $S_{vaep}$ swings from 0.830 to 0.177. VDEP is the more stable of the two across time horizons, which is what a metric describing a *team property* rather than a scoreline should do.

Note the metric inversion: VAEP scores better on **Brier**, VDEP on **AUC** and **F1**. See [[class-imbalance-evaluation]] — this is one of the cleaner demonstrations of why that choice matters.

## Limitations

- **Team-level only.** The authors are explicit: VDEP cannot rate individual defenders. The vault's individual-defender gap remains open.
- **$C \approx 3.9$ is arbitrary.** Frequency ratio encodes no judgement about whether one recovery is worth 3.9 prevented attacks. The authors call it controversial and defer it.
- **45 matches, one league, five weeks.** A pilot study, and labelled as one.
- **$k = 5$ unjustified beyond domain intuition**, with no sensitivity analysis — the same criticism this vault makes of [[temporal-discounting|Shelopugin's $\gamma$]] and Fernández et al.'s $\epsilon$.
- **Uncomparable comparison.** VDEP at $k=5$ against VAEP at $k=10$, predicting different events on a dataset much smaller than VAEP's original. This weakens "VDEP beats VAEP" while leaving intact the real claim, which is that goal-prediction classifiers fail on small data.

## Where It Sits

| | [[vaep]] | **VDEP** |
|---|---|---|
| Perspective | Attacking | **Defending** |
| Target | Score / concede | Recover / be attacked |
| Target frequency | 106 goals | 9,408 + 3,701 events |
| State | On-ball actions | On-ball **+ all 22 positions** |
| Unit | Player | **Team only** |
| Lookahead | $k = 10$ | $k = 5$ |

The two are complementary rather than competing, and the paper frames them that way — VDEP is proposed as an addition to existing indicators, not a replacement.

## See Also

- [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[vaep]] · [[action-valuation]] · [[off-ball-value]] · [[expected-goals]]
- [[shap]] · [[gradient-boosting]] · [[predictive-validity]]
- [[keisuke-fujii]] · [[kosuke-toda]]
- [[football-defence-evaluation-vdep|Source Summary]]
