---
title: "VDEP (Valuing Defense by Estimating Probabilities)"
type: concept
tags: [sports-analytics, defensive-valuation, action-valuation, off-ball, proxy-target, gradient-boosting, optical-tracking-data, evaluation, class-imbalance]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# VDEP (Valuing Defense by Estimating Probabilities)

[[football-defence-evaluation-vdep|Toda, Teranishi, Kushiro & Fujii's (2022)]] team-level defensive metric. Named and built as a deliberate counterpart to [[vaep]], modifying its published code, and differing on exactly two things: **what is predicted**, and **what the model can see**.

**Superseded in part by [[gvdep|GVDEP]]** (Umemoto, Tsutsui & Fujii, 2022), which fixes three limitations its authors named. See below.

## The Construction

Two classifiers over the next $k = 5$ events:

$$V_{vdep}(S_i) = P_{recoveries}(S_i) - C \cdot P_{attacked}(S_i)$$

- $P_{recoveries}$ — probability the defending team wins the ball back
- $P_{attacked}$ — probability the defending team concedes an *effective attack*
- $C \approx 3.9$, the observed frequency ratio of the two events

Team value is the mean of $V_{vdep}$ over that team's events in a match. XGBoost for both classifiers.

## The Two Departures from VAEP

**Frequent targets instead of rare ones.** VAEP predicts scoring and conceding. VDEP predicts recovery and being attacked, roughly 90× and 35× more frequent than goals. See [[rare-event-proxy-targets]].

**Off-ball state.** VDEP's state is $s_i = [a_i, o_i]$, where $o_i$ holds all 22 players' coordinates and their distances from the ball, **sorted by proximity** — making the representation permutation-invariant, so "nearest defender" occupies a fixed feature slot.

[[shap]] confirms the off-ball features dominate: nearest-defender distance is the top contributor to $P_{recoveries}$, and the nearest attacker's $x$-position to $P_{attacked}$.

## What "Effective Attack" Buys

The target is an effective attack — a chain ending in a shot **or** entering the penalty area — rather than a shot.

A forward who receives in the box and passes instead of shooting has still beaten the defence; scoring it as a non-event would reward defences for the attacker's choice. Defining failure as *territorial penetration* decouples the defensive assessment from attacking decision-making.

## Results

$P_{recoveries}$ F1 = 0.522, $P_{attacked}$ F1 = 0.484, against VAEP's $P_{scores}$ 0.201 and $P_{concedes}$ **0.000**.

⚠️ That last figure needs care. F1 = 0.000 is near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. [[gvdep|GVDEP]] reports concedes F1 of **0.08–0.15** on a different dataset, confirming the zero is not a fixed property of the model. See [[vaep-conceding-classifier]] and [[class-imbalance-evaluation]].

On correlations, $R_{vdep}$ relates similarly to match points (0.464) and season points (0.397), while $S_{vaep}$ swings from 0.830 to 0.177 — VDEP is the more stable across horizons, which is what a metric describing a *team property* should do.

## What GVDEP Fixed

[[gvdep|GVDEP]] addresses all three limitations Toda et al. named:

| VDEP limitation | GVDEP's fix |
|---|---|
| **$C$ from event frequency** — encodes how *often* each event happens, not how much each *matters* | Weight both terms by **[[vaep\|VAEP]] at the moments those events occur**, putting them on a score scale |
| **Assumed all 22 players observed** | Broadcast-frame data, plus a sweep showing ball gain saturates at **3–4 players** and the other three targets need none |
| **One domestic men's league** | Men's Euro 2020 and **women's Euro 2022** |

The $C$ fix matters beyond this metric: it is the only case in the vault of an asserted free parameter being **superseded by a principled derivation** rather than left unexamined. See [[model-selection]] and [[free-parameters-load-bearing]].

The n_nearest result also revises what VDEP's off-ball state was buying. VDEP fed all 22 positions; GVDEP shows **most of that was unnecessary** — and for the concedes classifier, actively harmful (F1 falls 0.15 → 0.08 as players are added).

**The gain/attacked trade-off replicates** across both papers — J-League and European tournament, men's and women's football.

## Limitations

- **Team-level only.** The authors are explicit: VDEP cannot rate individual defenders. GVDEP does not change this. See [[defensive-valuation]].
- **$C \approx 3.9$ is arbitrary** — superseded by GVDEP.
- **45 matches, one league, five weeks.** A pilot study, labelled as one.
- **$k = 5$ unjustified beyond domain intuition**, and inherited unchanged by GVDEP.
- **Uncomparable comparison.** VDEP at $k=5$ against VAEP at $k=10$, predicting different events on a much smaller dataset. Weakens "VDEP beats VAEP" while leaving intact the real claim, that goal-prediction classifiers struggle at this data scale.

## Where It Sits

| | [[vaep]] | **VDEP** | [[gvdep]] |
|---|---|---|---|
| Perspective | Attacking | **Defending** | Defending |
| Target | Score / concede | Recover / be attacked | Recover / be attacked |
| Weighting | — | **Frequency ratio** | **VAEP score scale** |
| Observation | On-ball | All 22 positions | **Broadcast frames** |
| Unit | Player | **Team** | **Team** |

## See Also

- [[gvdep]] · [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[vaep]] · [[vaep-conceding-classifier]] · [[action-valuation]] · [[off-ball-value]] · [[model-selection]]
- [[shap]] · [[gradient-boosting]] · [[predictive-validity]] · [[keisuke-fujii]] · [[kosuke-toda]]
- [[football-defence-evaluation-vdep|Source Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]]
