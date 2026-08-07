---
title: "Action-Space Design"
type: concept
tags: [action-space, reinforcement-learning, multi-agent, game-theory, counterfactual, action-valuation, off-ball, sports-analytics, feature-engineering, evaluation, simulator]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.75
provenance:
  extracted: 48%
  inferred: 35%
  generated: 16%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Action-Space Design

The choice of *what counts as an action* — and, where the underlying behaviour is continuous, how it is discretised. Usually presented as an implementation detail; it is in fact the single choice that determines which questions a valuation framework can answer.

## Why This Page Exists

[[reinforcement-learning]] carries a claim generated in this vault:

> **The barrier to optimal-policy analysis is the size of the action space, not the observational nature of the data.**

That was drawn from [[optimal-decisions-shot-taking-situations|Yeung & Fujii]], who make optimal-policy analysis tractable by shrinking the strategy space to {Shoot, Pass} × {Block, Not Block} — four profiles, all enumerable. [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] now provide a second data point at a different scale, which lets the claim be examined rather than merely repeated.

## Three Held Action Spaces

| Framework | Action space | Size | Counterfactuals available |
|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass destination — a **[[probability-surface\|surface]]** over the pitch | Effectively continuous | Every destination, by construction |
| [[optimal-decisions-shot-taking-situations\|Yeung & Fujii]] | {Shoot, Pass} × {Block, Not Block} | **4 profiles** | All four, by enumeration |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | 8 movement directions + idle + sprint start/stop + release + pass + shot | **14 per agent** | All 14, via learned $Q$ |

Nakahara et al.'s set is adapted from [[google-research-football|GFootball's]] 19 actions — a simulator's action vocabulary transplanted into an analysis of real matches.

## What Each Design Buys and Costs

**Continuous surface.** Values every option at full spatial resolution and never has to justify a discretisation. But it is defined over *destinations*, not over *what a player does* — so it can say where a pass should have gone and nothing about whether to pass at all, or about a player who is not passing. This is exactly the "targeting within an action" limitation identified on [[observed-versus-optimal-decisions]].

**Radical coarsening.** Makes [[game-theory|equilibrium]] computable and the answer explainable — the payoff table *is* the explanation. The cost is stated on [[reinforcement-learning]]: collapsing ten possible recipients into one "Pass" option means the equilibrium concerns a coarsened game, not football.

**Middle discretisation.** 14 actions is small enough that every action's value can be learned and reported, and large enough to cover movement, which is where off-ball value lives. It is the only one of the three that can value **a player who neither has the ball nor will receive it.**

## The Cost of the Middle, Which the Source Understates

Eight movement directions at 45° means **a run's direction is rounded to the nearest octant**, and its *magnitude* is not an action at all — speed enters only through three coarse labels (sprint start, sprint stop, release) gated by a 24 km/h threshold and a 0.1 m/s stop threshold.

Three things follow, none stated in the source:

1. **Two runs 40° apart may receive the same action label**, and thus the same counterfactual value, though their tactical meaning differs entirely.
2. **The action label is derived from realised velocity**, not from intent. A player who intended a run and was impeded is labelled by what happened, which is the [[intent-vs-outcome-valuation|intent/outcome]] problem relocated from outcomes to actions.
3. **The gates are unswept free parameters** that determine label assignment rather than value magnitude — see [[free-parameters-load-bearing]], where thresholds of this kind are the worst-behaved category.

There is also a straightforward omission: the dataset contains dribbling and trapping labels, discarded "for simplicity". So the on-ball action set is pass, shot and movement, which is narrower than [[spadl|SPADL]] and much narrower than the frameworks this paper compares itself against.

## The Generalisation

> ### `action-space-sets-the-question`
> **A framework's action space fixes which counterfactual questions it can pose, and therefore what "suboptimal" can mean within it. Frameworks with different action spaces are not measuring the same construct even when they report the same quantity.**
> ^[generated: drawn here across three held sources; none states it. Also on [[observed-versus-optimal-decisions]]. rests-on: source:fernandez-pass-surface, source:yeung-four-profiles, source:nakahara-14-actions]

This sharpens rather than contradicts the [[reinforcement-learning]] claim. Action-space size does govern *tractability* of optimal analysis. But size is not the only axis — **granularity and coverage matter independently**, and the three held frameworks differ on all three:

| | Size | Covers off-ball | Covers action *choice* |
|---|---|---|---|
| Fernández et al. | Huge | No | No |
| Yeung & Fujii | Tiny | No | **Yes** |
| Nakahara et al. | Middling | **Yes** | **Yes** |

No held framework covers all of: fine spatial granularity, off-ball players, and choice between action types. That is a real and specific gap, and it is a gap in the *field*, not merely in the vault — the three papers span the design space fairly completely.

## Consequence for Cross-Framework Comparison

This supplies a candidate explanation for the vault's most striking new number: **[[c-obso|C-OBSO]] and Nakahara's Q-values correlate at $\rho = 0.182$** on the same club, season and data.

C-OBSO's implicit action space is *the continuous trajectory a player ran*, valued against a predicted trajectory. Nakahara's is 14 discrete labels. Two metrics of "off-ball contribution" defined over different action spaces need not agree, and on this evidence do not. Whether that is a benign division of labour or a sign that at least one is mismeasuring is taken up on [[construct-validity]].

## Beyond Sport

Any system valuing decisions from logs faces the same choice, and usually makes it silently: which clinical interventions count as distinct, which trades are one action, which UI events are decisions. **The discretisation determines what "a better decision" could have meant**, and it is almost never reported as a modelling assumption.

## See Also

- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]]
- [[game-theory]] · [[probability-surface]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[policy-modelling]]
- [[action-valuation]] · [[off-ball-value]] · [[c-obso]] · [[spadl]] · [[intent-vs-outcome-valuation]] · [[construct-validity]]
- [[google-research-football]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[expected-value-possession-framework|EPV Summary]]
