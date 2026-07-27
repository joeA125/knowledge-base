---
title: "Expected Goals (xG)"
type: concept
tags: [sports-analytics, statistics, machine-learning, evaluation, action-valuation, player-evaluation, time-series, volatility, sample-weighting, gradient-boosting, optical-tracking-data, calibration]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
---

# Expected Goals (xG)

Expected Goals (xG) estimates the probability of a shot resulting in a goal, based on features of the shot opportunity (location, angle, body part, preceding action, defensive pressure, etc.).

Originally proposed for ice hockey (Macdonald, 2012) and adapted to football (Eggels, van Elk & Pechenizkiy, 2016). Both [[football-performance-time-series|Mendes-Neves et al.]] and [[epv-control-duel-skills-football|Shelopugin]] trace it further back, to Pollard, Ensum & Taylor (2004) on estimating goal probability from distance, angle and space.

## How It Works

For each goal attempt, an xG model predicts $P(\text{goal} \mid \text{shot features})$ using a classifier trained on historical shot data. A penalty might carry xG ≈ 0.76, a long-range effort ≈ 0.03. Summing a player's xG over a season gives their expected goal tally, enabling comparison with actual goals.

## xG Is an Intent Metric

xG is defined over the situation *at the moment of the shot* and deliberately excludes how well the ball was actually struck. It measures **chance quality, not strike quality**.

This makes it the clearest instance of [[intent-vs-outcome-valuation|intent-based valuation]], and explains the metric's most familiar debate. "Is he a good finisher or just getting good chances?" is precisely the intent/outcome question: goals minus xG is an outcome-versus-intent residual.

The I-VAEP/O-VAEP construction generalises the same split from shots to all 21 [[spadl]] action types.

## What Goes Into the Model

The feature set has expanded steadily with data availability, and the progression is instructive.

**Event-data xG** uses what an event stream records: location, distance, angle, body part, set-piece type, preceding action.

**Tracking-based xG** ([[expected-value-possession-framework|Fernández, Bornn & Cervone]]) adds what only [[optical-tracking-data|tracking]] can supply — the other 21 players:

- **Immediate pressure** — opponents within 3 metres of the shooter
- **Blockage count** — opponents inside the triangle formed by the shooter and the two posts, i.e. how interceptable the shot is
- **Goalkeeper geometry** — three features from the keeper's position, including whether the ball is closer to goal than the keeper
- Head versus foot

These are exactly the factors a viewer intuitively judges and an event stream cannot see. Two identical shot locations with a defender's leg across the line and with a clear sight of goal are not the same chance, and event-data xG scores them identically.

## Keeping the Model Player-Agnostic

If xG describes the *situation*, the model must not learn who is shooting. Shelopugin's implementation makes the necessary precautions explicit — both worth borrowing.

**Deliberate feature exclusion.** Current score, competition, and shooting team are omitted *because they correlate with player skill*. A model told "Manchester City, 2023/24" has been handed a proxy for shot quality unrelated to the chance's geometry.

**[[sample-weighting|Frequency-weighted loss]].** Elite attackers take vastly more shots, so an unweighted fit partly learns their finishing and reports it as chance quality. Dividing each instance's log-loss by the player's frequency equalises players rather than events. The cost is variance.

**Separate set-piece and open-play models.** Set-piece xG is fit independently, since a penalty or free kick should depend only on the current situation, whereas open-play xG legitimately depends on preceding actions.

## xG as Its Own Prior

An arrangement worth noting, because it inverts the usual data hierarchy.

Fernández et al. hold 13,735 shots with tracking data — but 117,948 shots with event data only, nearly nine times as many. Rather than choose, they train a **baseline event-data xG** on the large set (XGBoost; log loss 0.2540, ECE 0.00594) and feed its output as an *input feature* to the richer tracking-based model.

So the coarse model supplies a well-calibrated prior that the fine model refines with spatial context. The tracking model never has to relearn the basic distance-and-angle relationship from a small sample; it only has to learn the correction that pressure, blockage and keeper position imply.

This is a clean pattern for any domain where a large coarse dataset coexists with a small rich one, and it is more robust than the obvious alternative of training only on the rich data.

## A Building Block, Not Just a Competitor

xG is not merely superseded by broader frameworks; it is a *component* of them.

- In [[vaep]], xG is a special case: computing a shot's xG equals estimating $P_{scores}$ at the state just before it.
- In [[expected-threat|xT]], xG appears inside the value recursion: $xT(z) = s_z \cdot xG(z) + m_z \sum_{z'} T_{z \to z'} xT(z')$.
- In [[expected-possession-value|Shelopugin's EPV]], accumulated future xG *is* the training target — the binary "did this possession end in a goal" label is rejected because goals are too rare and would overfit.
- In Fernández et al.'s decomposition, xG *is* the shot branch: $\mathbb{E}[G \mid A = \varsigma, T_t]$ is an expected goals model by definition.

The third use inverts the usual complaint. xG is normally criticised for covering too few events; here its role is to **supply density** where raw goals cannot. A model targeting goals sees ~2.6 positive events per match; one targeting accumulated xG sees a real-valued signal at every shot.

## Limitations

The [[evaluating-football-player-actions|VAEP paper]] identifies three:

1. **Shot-centric** — values only the final shot, ignoring the build-up.
2. **Context-blind** — fixed values by location, ignoring game state. *(Partly addressed by tracking-based variants above.)*
3. **Immediate only** — no longer-term effects of actions several steps before a shot.

The underlying issue is sparsity. Shots and assists together are **less than 1% of on-the-ball actions**; in the WyScout data used by [[nmstpp]], shots are 1.68% of events. xG is undefined for the other 98%, which motivates the whole [[action-valuation]] literature.

A fourth, from the same sparsity: xG for a *single player over a short window* is extremely noisy. This is the rarity problem that drives [[split-half-reliability|VAEP's low reliability]] and that [[performance-volatility|volatility analysis]] must control for. It also shows up inside Fernández et al.'s own results — shot EPV has by far the worst loss of any component (0.2421), which the authors attribute directly to sample size.

## Can Team Performance Be Assessed Without It?

[[hpus]] provides a test. Derived from [[nmstpp]] forecasts, it uses **no goal or shot-outcome data at any stage** — yet correlates **0.92 with season xG** and −0.78 with final league position (against xG's −0.81).

That a metric built purely from event *dynamics* recovers most of xG's signal suggests the two measure substantially overlapping things by different routes.

## See Also

- [[action-valuation]] · [[expected-possession-value]] · [[intent-vs-outcome-valuation]]
- [[vaep]] · [[expected-threat]] · [[hpus]] · [[pass-carry-reward]]
- [[spadl]] · [[performance-volatility]] · [[probability-calibration]]
- [[sample-weighting]] · [[gradient-boosting]] · [[optical-tracking-data]]
- [[structured-model-decomposition]] · [[action-valuation-frameworks-compared]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]]
- [[football-performance-time-series|Valuing Players Over Time Summary]] · [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
- [[expected-value-possession-framework|Soccer EPV Framework Summary]]
