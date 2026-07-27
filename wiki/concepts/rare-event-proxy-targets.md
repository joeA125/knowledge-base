---
title: "Rare-Event Proxy Targets"
type: concept
tags: [proxy-target, class-imbalance, machine-learning, statistics, evaluation, sports-analytics, defensive-valuation, predictive-validity]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
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

| Framework | Rare target abandoned | Frequent proxy adopted | Ratio |
|---|---|---|---|
| [[vdep]] | Goals conceded | Ball recovery, effective attack | ~90×, ~35× |
| [[expected-goals\|xG-as-target]] | Goal scored | Accumulated shot xG | Real-valued at every shot |
| [[hpus]] | Goals | Possession utilisation from event dynamics | Uses no goal data at all |
| [[nmstpp]] | — | Next event time, zone, action | Every event is a label |

[[epv-control-duel-skills-football|Shelopugin]] states the reasoning explicitly, rejecting a binary "did this possession end in a goal" target as too sparse and overfitting-prone, and using accumulated future xG instead. Same move, different proxy.

## What Makes a Proxy Good

Three conditions, and the second is the one that gets skipped.

**1. Frequent enough to learn from.** The point of the exercise.

**2. Causally upstream of the real outcome, not merely correlated.** A proxy that co-occurs with goals for incidental reasons will be optimised in ways that do not produce goals. Ball recovery and territorial penetration sit on the causal path to conceding; a metric like "passes completed" correlates with winning largely through confounding.

**3. Not so far upstream that it measures something else.** Push far enough and the proxy becomes its own construct with its own validity question.

The strongest available evidence that the move works comes from [[hpus]], which uses **no goal or shot-outcome data at any stage** yet correlates 0.92 with season xG and −0.78 with league position, against xG's −0.81. A metric built purely from event dynamics recovers nearly all the signal of one built from outcomes.

## The Cost

**The proxy becomes the definition.** [[vdep]] does not measure defensive quality; it measures recovery-and-penetration performance, which is a *hypothesis about* defensive quality. If the hypothesis is wrong, the metric is confidently wrong. The authors are appropriately careful, noting they cannot validate against ground truth.

**Weighting is arbitrary.** Combining proxies needs relative weights, and there is rarely a principled source for them. VDEP's $C \approx 3.9$ comes from the observed frequency ratio — which encodes *how often* each event happens, not *how much each matters*. The authors call it controversial and defer it. This is the general shape of the problem: frequency is available, importance is not.

**Validation gets harder, not easier.** You can no longer check against the real outcome without reintroducing the sparsity you escaped. [[predictive-validity]] against downstream results becomes the main available test — and VDEP's season-points correlation of 0.397 is moderate, not strong.

**Goodhart risk.** A proxy optimised as a target drifts from the thing it proxied. Acute in sport, where a team told to maximise ball recoveries can do so by conceding territory cheaply.

## Elsewhere

The pattern is standard wherever the outcome of interest is rare and costly:

- **Medicine** — surrogate endpoints (tumour shrinkage for survival, viral load for progression), with a long literature on surrogates that failed to track the real outcome.
- **Reliability engineering** — near-misses and precursor events instead of failures.
- **Safety** — leading indicators instead of accident counts.
- **Advertising** — clicks instead of purchases.

The medical literature is the cautionary one: surrogate endpoints have repeatedly been validated, adopted, and later found not to predict the outcome that mattered. Condition 2 above is where those failures live.

## See Also

- [[class-imbalance-evaluation]] · [[vdep]] · [[defensive-valuation]]
- [[expected-goals]] · [[hpus]] · [[vaep]] · [[pass-carry-reward]]
- [[predictive-validity]] · [[probability-calibration]]
- [[football-defence-evaluation-vdep|Source Summary]]
