---
title: "Construct Validity"
type: concept
tags: [construct-validity, evaluation, predictive-validity, reliability, player-evaluation, sports-analytics, action-valuation, off-ball, statistics, needs-review]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md, raw/papers/football-performance-time-series.md]
confidence: 0.7
provenance:
  extracted: 42%
  inferred: 33%
  generated: 20%
  imported: 5%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Construct Validity

Whether a metric measures the thing it claims to. Judged not by accuracy against ground truth — there usually is none — but by the **pattern** of its agreement and disagreement with other metrics.

Two halves, and the second is the one this literature keeps stumbling into:

- **Convergent** — the metric agrees with other measures of the *same* construct.
- **Discriminant** — the metric *diverges* from measures of *different* constructs.

## Why It Matters Here Specifically

Every off-ball metric in this vault has the same problem: **there is no ground truth for off-ball contribution.** Nobody observes the true value of a run. [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] state it flatly — since there is no true Q-value, model performance cannot be directly validated.

So the field validates by correlation with whatever else exists. That is construct validation whether or not anyone calls it that, and it is worth having the vocabulary, because **the same correlation can be reported as a success or a failure depending on which half you invoke**, and papers here choose without arguing for the choice.

## The Move That Prompted This Page

Nakahara et al. report $\rho = -0.761$ between their metric and season goals, and present this as evidence the method works.

The argument is discriminant: goals measure finishing, the metric measures off-ball and distributive contribution, so a *negative* relationship confirms it is picking up what goal-counting misses. Their ranking is topped by two defenders with zero goals between them, one of whom was second in the league for passes.

That argument is legitimate. It is also **unfalsifiable as deployed**, which is the problem.

> ### `discriminant-claims-need-a-convergent-anchor`
> **A metric validated only by divergence from existing measures cannot be distinguished from a metric measuring nothing. Noise also diverges from goals.**
> ^[generated: no source states this; drawn here from Nakahara et al.'s validation strategy. rests-on: source:nakahara-negative-goal-correlation, source:nakahara-no-ground-truth]

A metric needs at least one thing it is *supposed* to agree with, specified in advance. Nakahara et al. offer none: they diverge from goals (−0.761), from expert ratings (−0.218), from [[obso|OBSO]] (−0.305), and are unrelated to [[c-obso|C-OBSO]] (0.182). **Every comparison they report is a divergence, and all four are read as favourable.**

Their strongest available convergent evidence is not a correlation at all but a face-validity check: Thiago and Hatanaka, both highly ranked, were both among the league's top passers, and Thiago won a league best-XI award. That is real evidence. It is also two players.

## The $\rho = 0.182$ Problem

The C-OBSO comparison is the sharpest case in the vault, because here the discriminant reading is much harder to sustain.

[[c-obso|C-OBSO]] and Nakahara's Q-values are both from the Fujii group, both computed on **Yokohama F. Marinos, J1 2019, [[data-stadium|Data Stadium]] tracking**, both explicitly presented as measures of **off-ball contribution**. They correlate at 0.182.

| Reading | Claim | Status |
|---|---|---|
| **Benign** (the paper's) | They measure different *aspects* — C-OBSO the creator, Q-values the distributor and mover | Plausible; supported by which players each favours |
| **Harsh** (not considered) | "Off-ball contribution" is not one construct, so at least one metric is named more broadly than it measures | Equally consistent with the evidence |

The paper's own explanation concedes most of the harsh reading. If C-OBSO ranks forwards and Q-values rank midfielders and defenders, then neither measures off-ball contribution *as such* — each measures a positional slice of it. **The honest report would rename both**, or state the construct as position-conditional.

**A third explanation exists and is more mundane.** The four held sources on this dataset each **subset it differently** — C-OBSO uses shot-ending sequences only, Nakahara et al. attacking-third possessions regardless of outcome. Metrics computed over different populations of moments need not agree, and no comparison controls for this. See [[data-stadium]].

This is the vault's first head-to-head between two off-ball metrics, and it is not reassuring. See [[off-ball-value]].

## The Three Validity Criteria, Compared

| | [[split-half-reliability\|Reliability]] | [[predictive-validity]] | **Construct validity** |
|---|---|---|---|
| Asks | Is it stable? | Does it forecast? | Is it *the thing*? |
| Needs | Repeated samples | A future outcome | Other metrics |
| Can be gamed by | A degenerate constant | A proxy for the outcome | **Divergence from everything** |
| Reported by | [[player-rating-time-series\|Mendes-Neves et al.]], [[on-ball-actions-football-xt-vs-vaep\|Van Roy et al.]] | [[obso\|OBSO]] (0.26 next-match goals) | Nakahara et al., [[c-obso\|C-OBSO]] |

These are not substitutes and none is sufficient alone. The strongest result in the vault — OBSO predicting next-match goals at 0.26, beating shots and goals themselves — is *predictive*, against an external outcome, and that is why it is the strongest.

**No held source reports all three for one metric.** Nakahara et al. report construct evidence only; [[c-obso|C-OBSO]] reports construct evidence (salary, expert ratings) only; neither reports reliability, consistent with the standing absence claim on [[off-ball-value]].

## The Salary and Expert-Rating Route

Both papers reach for external quantities as stand-ins for ground truth.

- **Annual salary** ([[c-obso|C-OBSO]], $\rho = 0.45$) — reflects the market's judgement, confounded by age, position, nationality and contract timing.
- **Expert match ratings** (both papers) — reflect journalists' judgement, and are known from C-OBSO's own tables to be **heavily goal-driven**.

Neither is ground truth; both are *other raters*. Agreement is convergent evidence against a noisy human criterion, and disagreement is ambiguous between "the metric sees what humans miss" and "the metric is wrong". C-OBSO's salary result is the more useful mainly because it was the only positive result among three tested on one sample, which makes it harder to attribute to fishing.

## What Would Settle It

1. **Specify a convergent target in advance.** For a metric claiming to value distribution and movement, pass volume, progressive distance carried, or [[space-occupation-gain|SOG]] are all candidates. Nakahara et al. *observe* their top players are top passers — pre-registering that as the test would convert an anecdote into evidence.
2. **Report reliability alongside.** A metric that diverges from everything *and* is unstable across splits is noise. This distinguishes the two readings above and nothing else does.
3. **Compare on identical samples** — and on identical *subsets*. Already possible: C-OBSO, the Q-values, [[obso|OBSO]] and [[space-occupation-gain|SOG]] could be computed on the same 14 players over the same possessions, and the correlation matrix would say whether "off-ball value" has one factor or several.

The third is the closest thing to a decisive, cheap test the vault has identified for off-ball valuation, and no source has run it.

## Beyond Sport

Wherever a construct is unobservable — job performance, model capability, patient wellbeing, code quality — validation reduces to a correlation matrix among instruments. The failure mode is universal: **a new instrument that agrees with nothing is reported as measuring something new, when it may be measuring nothing.** See [[capability-profiling]] for the same problem in AI evaluation, where reporting a vector rather than a composite is one response to it.

## See Also

- [[predictive-validity]] · [[split-half-reliability]] · [[capability-profiling]] · [[uncertainty-quantification]] · [[probability-calibration]]
- [[off-ball-value]] · [[c-obso]] · [[obso]] · [[space-occupation-gain]] · [[action-valuation]] · [[action-space-design]] · [[data-stadium]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[selection-bias]] · [[recruitment]] · [[dynamic-time-warping]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] · [[beyond-expected-goals|OBSO]]
