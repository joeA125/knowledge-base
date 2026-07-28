---
title: "Evaluation of Creating Scoring Opportunities for Teammates in Soccer via Trajectory Prediction (C-OBSO)"
type: source_summary
tags: [summary, sports-analytics, off-ball, space-creation, player-evaluation, trajectory-prediction, counterfactual, optical-tracking-data, pitch-control, graph-neural-network, vae, deep-learning, evaluation]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Evaluation of Creating Scoring Opportunities for Teammates in Soccer via Trajectory Prediction

**Authors:** [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-fujii]] (senior)
**Venue:** International Workshop on Machine Learning and Data Mining for Sports Analytics (MLSA), Springer, pp. 53–73. Cited variously as 2022 (workshop) and 2023 (proceedings).
**Code:** github.com/keisuke198619/C-OBSO

Acquired following a [[defensive-valuation|correction]] in which this paper was identified as cited-but-not-held. It closes a gap this vault had listed as open across three pages: **credit for creating space that a teammate exploits.**

## The Problem

The framing statistic is the one every off-ball paper opens with: **a player has the ball for roughly 3 of 90 minutes.** Everything else is off-ball, continuous, undiscretised, and — for most analytics — invisible.

Existing off-ball work values the player who would *receive* the ball ([[obso|OBSO]]) or the space created in the abstract (Fernández & Bornn's "Wide Open Spaces"). Neither answers: how much did *this* player's movement improve the scoring chance of *a different* player?

## C-OBSO

**Creating Off-Ball Scoring Opportunity.** For an off-ball player $i$ and the eventual ball carrier $k$:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

where $V^k_{OBSO}$ is the shooter's [[obso|OBSO]] in the situation that actually occurred, and $V'^k_{OBSO}$ is the shooter's OBSO in a counterfactual where player $i$ moved as a **predicted reference trajectory** says an average player would have.

The credit assigned to $i$ is therefore *the improvement in someone else's scoring chance attributable to $i$ deviating from expected movement.* In the paper's worked example, A1 draws his marker D1 further than predicted, opening the shooter A2 — and C-OBSO is positive.

This is a [[counterfactual-baseline|predicted-reference baseline]], and the construction has a notable property the authors state plainly: **if prediction were perfect, C-OBSO would always be zero.** The metric depends on the predictor being imperfect, which is unusual and worth thinking about carefully.

## Three Components

### 1. A better score model inside OBSO

[[obso|OBSO]] factorises into control × transition × score. The original score term $P(S_r|D)$ is a function of distance to goal alone.

The **potential score model** replaces it with one reflecting **angle to goal and the positions of multiple defenders**. Shot value is computed per-degree across the shooting angle, discounted by a shot-blocking distribution: a Gaussian per defender goal-side of the shooter, variance widening with distance, and **goalkeepers weighted double**.

$$P(S^p_r|D) = \sum_{i=1}^{n} V_{shot}(\vec{s}_i), \qquad V_{shot}(\vec{s}) = C(c - V_{block})$$

Validated on 494 Yokohama shots (59 goals): RMSE **0.309** against the distance-only model's **0.324** ($p < 10^{-10}$). Modest but real. The illustrative case is two shots from near-identical distance — identical under the old model (both 0.1237), separated under the new one (0.0489 vs 0.1202) by defender congestion.

### 2. Trajectory prediction as the reference

A **[[trajectory-prediction|graph variational recurrent neural network]]** (GVRNN) generates the counterfactual movement. Trained on *opponent* data to produce "league average" trajectories, then applied to Yokohama.

Endpoint error (MAE, metres) against baselines:

| | 4 s | 6 s | 8 s | 10 s |
|---|---|---|---|---|
| **GVRNN** | **0.608** | **0.867** | 1.701 | 1.606 |
| VRNN | 5.952 | 7.767 | 9.127 | 10.168 |
| RNN+Gauss | 9.101 | 11.396 | 13.312 | 15.327 |

An order of magnitude. The graph structure — modelling players as nodes with learned pairwise edges — is doing almost all the work, which is unsurprising given that a player's next position depends overwhelmingly on the others.

4-second prediction is used for C-OBSO: long enough for movement to differentiate, short enough for MAE below 0.7 m.

### 3. The difference

Only **three players** are predicted simultaneously: one off-ball attacker and the defenders nearest each of two attackers. The authors are explicit that this is a computational limit, not a modelling choice — error grows with the number of agents predicted.

## Data

34 games of **Yokohama F. Marinos**, Meiji J1 League 2019, from [[data-stadium|Data Stadium Inc.]] — the same provider, league and season as [[football-defence-evaluation-vdep|VDEP]]. Tracking at 25 Hz, events at 30 Hz. Training on opponents, testing on Yokohama.

## Validation, and Its Difficulty

No ground truth exists, so three confounded proxies are used.

**Annual salary** (15 players, ≥10 evaluated sequences):

| Metric | ρ with salary | p |
|---|---|---|
| **C-OBSO** | **0.45** | **0.046** |
| [[obso\|OBSO]] | −0.28 | 0.154 |
| Goals | −0.23 | 0.208 |

C-OBSO is the only one of the three that relates to salary at all. Neither a player's own off-ball scoring opportunity nor his goal tally does — which is a striking negative result in its own right, and the paper's strongest argument.

The two players well above the salary trend line (in red in Fig. 3A) both won individual awards that season, and both had large pay rises the following year (11M → 40M yen; 20M → 60M yen). Suggestive rather than evidential, but the right direction.

**Expert match ratings**, top three scorers:

| Player | ρ (C-OBSO vs rating) | ρ (goals vs rating) |
|---|---|---|
| Nakagawa (MVP) | **0.75** (p = 0.0003) | 0.63 |
| Marcos | 0.27 (ns) | 0.71 |
| Edigar | −0.37 (ns) | 0.91 |

This table is more interesting than the paper makes it. **Goals predict expert ratings strongly for every player; C-OBSO predicts them only for the season's MVP.** The natural reading is that match ratings are largely goal-driven, and C-OBSO measures something the raters mostly do not reward — except for one player whose reputation rested on more than finishing.

That is either evidence the metric captures something real that ratings miss, or evidence it is noise that happened to align once. Seven further players show no correlation. The authors report this honestly and do not over-claim.

## Assessment

**Strengths.** The construction is elegant and the credit assignment is genuinely *relational* — value flows from the mover to the shooter, which no other framework in this vault expresses. The GVRNN margin over baselines is large and clearly reported. The salary result is the only positive finding among three metrics tested on the same players, which makes it harder to dismiss as fishing. Code is public.

**Weaknesses.**

- **Negative C-OBSO is clipped to zero.** Justified on the grounds that the *predicted* defender often fails to take a sensible position, so negatives reflect predictor error rather than player error. But it means the metric cannot penalise bad movement, and it introduces a floor that inflates every player's mean.
- **The metric depends on the predictor being wrong.** Perfect prediction gives identically zero. So C-OBSO measures deviation from a *particular model's* expectation, and improving the trajectory model would shrink the metric — an uncomfortable dependency the authors acknowledge only obliquely.
- **Values are tiny** (0.001–0.01) and on no interpretable scale. The authors name this as future work.
- **Three predicted players out of 22**, for computational reasons. Fujii has separately described the full-22 version as prohibitively expensive.
- **One team, 34 games, one season.** Training on opponents and testing on Yokohama is a sensible split given the data, but the result is a study of one squad.
- **Salary is heavily confounded** — by age, position, nationality, contract timing and reputation. The authors say so.
- **No [[split-half-reliability|reliability]] figure**, as with every other metric in this vault.

## Why It Matters Here

**It closes the space-creation gap.** [[off-ball-value]], [[action-valuation]] and [[defensive-valuation]] all listed "credit for creating space someone else exploits" as unaddressed. This addresses it directly, at individual-player level, from a held primary source.

**It supplies the vault's first primary account of [[obso|OBSO]]**, including the PPCF formulation, which had appeared only as a cited dependency of the Fujii and Fernández lines.

**It is a different counterfactual from the vault's others.** [[scoutgpt]] substitutes an entity and regenerates; [[eventgpt]] substitutes and re-scores. This substitutes *nothing* — it compares the actual world against a predicted one, and attributes the difference. See [[counterfactual-baseline]].

## See Also

- [[c-obso]] · [[obso]] · [[counterfactual-baseline]] · [[trajectory-prediction]]
- [[off-ball-value]] · [[pitch-control]] · [[space-creation|space creation]] · [[defensive-valuation]]
- [[graph-neural-network]] · [[variational-autoencoder]] · [[expected-goals]]
- [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-fujii]] · [[william-spearman]]
- [[nagoya-university]] · [[data-stadium]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[action-valuation-frameworks-compared]]
