---
title: "Rare-Event Proxy Targets"
type: concept
tags: [proxy-target, class-imbalance, machine-learning, statistics, evaluation, sports-analytics, defensive-valuation, predictive-validity, game-theory]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Rare-Event Proxy Targets

When the outcome you care about is too rare to learn from, train on a **frequent event on its causal path** instead. A modelling move that appears independently in several vault sources and deserves naming, because it is usually presented as a domain trick rather than the general method it is.

## The Problem

Supervised learning needs positives. When the positive class is a fraction of a percent, three things go wrong at once:

- **Too little signal.** A few hundred positives cannot support a rich feature space.
- **Degenerate optima.** Predicting "no" always is nearly always right, and gradient descent finds that.
- **Misleading metrics.** Accuracy, Brier and AUC are all inflated by true negatives. See [[class-imbalance-evaluation]].

Football is a severe case. In the [[football-defence-evaluation-vdep|VDEP dataset]], 45 matches produced **106 goals** across 97,335 events. The consequence is measured, not asserted: [[vaep]]'s conceding classifier achieves **F1 = 0.000** — it identifies no true positives whatsoever.

## The Move

Replace the target with a frequent correlate:

| Framework | Rare target abandoned | Frequent proxy adopted | Gain |
|---|---|---|---|
| [[vdep]] | Goals conceded | Ball recovery, effective attack | ~90×, ~35× more frequent |
| [[xsot\|Yeung & Fujii]] | Goal | **Shot on target** | ~1 in 3 shots, not ~1 in 10 |
| [[pass-carry-reward\|Shelopugin]] | "Possession ended in a goal" (binary) | Accumulated future xG | Real-valued at every shot |
| [[hpus]] | Goals | Possession utilisation from event dynamics | **No goal data at all** |
| [[obso\|Spearman]] | Goal, as a *rating* | Positional opportunity | Every frame, not every goal |

[[epv-control-duel-skills-football|Shelopugin]] states the reasoning explicitly, rejecting a binary goal target as too sparse and overfitting-prone. [[optimal-decisions-shot-taking-situations|Yeung & Fujii]] frame theirs as **the minimum requirement of a good shot** — if it is not on target it cannot score.

## What a Proxy Change Does to the Model

An underrated consequence, and Yeung & Fujii supply the cleanest illustration: **changing the target changes which parts of the world matter.**

Because their payoff is shot-*on-target* rather than goal, a save still counts as success — so **the goalkeeper drops out of the shot-block model entirely** and is filtered from the defender set. In [[c-obso|C-OBSO]], whose target is a goal, the goalkeeper is included and weighted double.

Same sport, same geometry, opposite treatment of one player, entirely because the target moved. A proxy is not a neutral substitution for the outcome; it reorganises the model around itself.

## What Makes a Proxy Good

Three conditions, and the second is the one that gets skipped.

**1. Frequent enough to learn from.** The point of the exercise.

**2. Causally upstream of the real outcome, not merely correlated.** A proxy that co-occurs with goals for incidental reasons will be optimised in ways that do not produce goals. Ball recovery, territorial penetration and shot-on-target all sit on the causal path; "passes completed" correlates with winning largely through confounding.

**3. Not so far upstream that it measures something else.** Push far enough and the proxy becomes its own construct with its own validity question.

## The Evidence That It Works

Three independent results, all pointing the same way:

- [[hpus]] uses **no goal or shot-outcome data at any stage** yet correlates 0.92 with season xG and −0.78 with league position, against xG's −0.81.
- [[xsot|xSOT]] correlates **0.58** with average goals against [[expected-goals|xG]]'s **0.46** across World Cup 2022 teams.
- [[obso|OBSO]] predicts a player's next-match goals (0.26) better than his shots (0.17) or goals (0.12) do.

The pattern is consistent: **a metric built on a denser proxy can outperform one built on the outcome itself**, because the outcome is a noisy realisation of the process both are trying to measure. See [[predictive-validity]].

## The Cost

**The proxy becomes the definition.** [[vdep]] does not measure defensive quality; it measures recovery-and-penetration performance, which is a *hypothesis about* defensive quality. If the hypothesis is wrong, the metric is confidently wrong.

**Weighting is arbitrary.** Combining proxies needs relative weights, and there is rarely a principled source. VDEP's $C \approx 3.9$ comes from the observed frequency ratio — which encodes *how often* each event happens, not *how much each matters*. Frequency is available; importance is not.

**Validation gets harder.** You can no longer check against the real outcome without reintroducing the sparsity you escaped. [[predictive-validity]] against downstream results becomes the main available test.

**Goodhart risk.** A proxy optimised as a target drifts from the thing it proxied. Acute in sport, where a team told to maximise ball recoveries can do so by conceding territory cheaply.

## Elsewhere

Standard wherever the outcome of interest is rare and costly: surrogate endpoints in medicine (tumour shrinkage for survival), near-misses in reliability engineering, leading indicators in safety, clicks instead of purchases in advertising.

The medical literature is the cautionary one: surrogate endpoints have repeatedly been validated, adopted, and later found not to predict the outcome that mattered. Condition 2 above is where those failures live.

## See Also

- [[class-imbalance-evaluation]] · [[vdep]] · [[xsot]] · [[defensive-valuation]]
- [[expected-goals]] · [[hpus]] · [[obso]] · [[vaep]] · [[pass-carry-reward]]
- [[predictive-validity]] · [[probability-calibration]] · [[game-theory]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]]
