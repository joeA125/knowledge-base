---
title: "Expected Goals (xG)"
type: concept
tags: [sports-analytics, statistics, machine-learning, evaluation, action-valuation, player-evaluation, time-series, volatility, sample-weighting, gradient-boosting, optical-tracking-data, calibration, proxy-target, theory-based-modelling]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

Expected Goals estimates the probability of a shot resulting in a goal, from features of the shot opportunity (location, angle, body part, preceding action, defensive pressure).

Proposed for ice hockey (Macdonald, 2012) and adapted to football (Eggels, van Elk & Pechenizkiy, 2016), with the lineage traced back to Pollard, Ensum & Taylor (2004) on estimating goal probability from distance, angle and space.

## xG Is an Intent Metric

xG is defined over the situation *at the moment of the shot* and deliberately excludes how well the ball was struck. It measures **chance quality, not strike quality**.

This makes it the clearest instance of [[intent-vs-outcome-valuation|intent-based valuation]], and explains the familiar debate: "is he a good finisher or just getting good chances?" is precisely the intent/outcome question, and goals minus xG is the residual.

## Three Formulations, Never Benchmarked

The vault holds three ways of computing shot value, and **no source compares them**:

| | Mechanism | Inputs |
|---|---|---|
| **Event-data logistic/boosted** | Learned classifier | Location, angle, body part, preceding action |
| **Tracking-augmented** ([[expected-value-possession-framework\|Fernández et al.]]) | Learned classifier | Adds pressure count, blockage triangle, three goalkeeper features |
| **Angular-geometric** ([[obso\|OBSO]], [[c-obso]]) | [[theory-based-modelling\|Physical model]] | Per-degree integration over shot angle, discounted by defender occlusion |

The third is not usually recognised as an xG model, but that is what it is — and it is fitted with a handful of parameters rather than learned, which is why [[obso|Spearman]] can estimate it from five training matches.

## What Goes Into the Model

**Event-data xG** uses what an event stream records: location, distance, angle, body part, set-piece type, preceding action.

**Tracking-based xG** adds what only [[optical-tracking-data|tracking]] supplies — the other 21 players:

- **Immediate pressure** — opponents within 3 metres
- **Blockage count** — opponents inside the triangle formed by shooter and posts
- **Goalkeeper geometry** — three features, including whether the ball is closer to goal than the keeper
- Head versus foot

These are exactly the factors a viewer judges intuitively and an event stream cannot see. Two identical locations, one with a defender's leg across the line, are not the same chance.

## Keeping the Model Player-Agnostic

If xG describes the *situation*, the model must not learn who is shooting. Two precautions, both from [[epv-control-duel-skills-football|Shelopugin]]:

**Deliberate feature exclusion.** Score, competition and shooting team are omitted *because they correlate with player skill*.

**[[sample-weighting|Frequency-weighted loss]].** Elite attackers take vastly more shots, so an unweighted fit partly learns their finishing and reports it as chance quality. Dividing each instance's log-loss by the player's frequency equalises players rather than events. The cost is variance.

**Separate set-piece and open-play models**, since a penalty depends only on the current situation while open-play xG legitimately depends on preceding actions.

## xG as Its Own Prior

An arrangement that inverts the usual data hierarchy. [[expected-value-possession-framework|Fernández et al.]] hold 13,735 shots with tracking but 117,948 with event data only. Rather than choose, they train a **baseline event-data xG** on the large set (XGBoost; log loss 0.2540, ECE 0.00594) and feed its output as an *input feature* to the tracking model.

The coarse model supplies a well-calibrated prior; the fine model learns only the correction that pressure and occlusion imply. A reusable pattern wherever a large coarse dataset coexists with a small rich one — and the first of two vault instances of [[theory-based-modelling|hybrid modelling]].

## Being Outperformed by Denser Proxies

The most consistent finding about xG in this vault is that **metrics built on more frequent events predict goals better than xG does.**

| Comparison | Denser metric | xG |
|---|---|---|
| Correlation with average goals, World Cup 2022 teams ([[xsot]]) | **0.58** | 0.46 |
| Correlation with next-match goals, player level ([[obso]]) | **0.26** | — (goals 0.12, shots 0.17) |
| Correlation with season xG, using no goal data at all ([[hpus]]) | **0.92** | — |

[[xsot|xSOT]] is the sharpest case, because it is the *same kind of object* — a per-shot expectation — differing only in target. Substituting *shot on target* for *goal* raises the correlation with actual goals by 0.12.

The explanation is the one running through [[rare-event-proxy-targets]]: goals are a noisy realisation of a process, and a metric estimating that process from denser evidence can track it better than one estimating it from the sparse outcome. xG is itself an early instance of this logic — it exists because goals are too rare to rate a player on — and the later work simply pushes the same move further down the causal chain.

## A Building Block, Not Just a Competitor

xG is a *component* of nearly everything else here:

- In [[vaep]], computing a shot's xG equals estimating $P_{scores}$ just before it.
- In [[expected-threat|xT]] it sits inside the value recursion: $xT(z) = s_z \cdot xG(z) + m_z \sum_{z'} T_{z \to z'} xT(z')$.
- In [[expected-possession-value|Shelopugin's EPV]], accumulated future xG *is* the training target — chosen because binary goal labels are too sparse.
- In Fernández et al.'s decomposition, the shot branch $\mathbb{E}[G \mid A = \varsigma, T_t]$ **is** an xG model by definition.
- In [[obso|OBSO]], the score term is a geometric xG.

## Limitations

The [[evaluating-football-player-actions|VAEP paper]] identifies three: **shot-centric** (ignores build-up), **context-blind** (partly addressed by tracking variants), and **immediate only**.

The underlying issue is sparsity. Shots and assists are **less than 1% of on-the-ball actions**; in [[nmstpp|NMSTPP's]] WyScout data, shots are 1.68% of events.

A fourth, from the same source: xG for a *single player over a short window* is extremely noisy. This drives [[split-half-reliability|VAEP's low reliability]] and shows up inside Fernández et al.'s own results, where shot EPV has by far the worst component loss (0.2421), attributed directly to sample size.

## See Also

- [[action-valuation]] · [[expected-possession-value]] · [[intent-vs-outcome-valuation]] · [[xsot]]
- [[vaep]] · [[expected-threat]] · [[hpus]] · [[obso]] · [[c-obso]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[theory-based-modelling]] · [[probability-calibration]]
- [[spadl]] · [[sample-weighting]] · [[gradient-boosting]] · [[optical-tracking-data]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[expected-value-possession-framework|Soccer EPV Summary]]
- [[beyond-expected-goals|Spearman Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
