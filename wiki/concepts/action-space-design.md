---
title: "Action-Space Design"
type: concept
tags: [action-space, reinforcement-learning, multi-agent, game-theory, counterfactual, action-valuation, off-ball, sports-analytics, feature-engineering, evaluation, simulator, domain-adaptation]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/expected_value_possession_framework.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.8
provenance:
  extracted: 52%
  inferred: 32%
  generated: 15%
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

That was drawn from [[optimal-decisions-shot-taking-situations|Yeung & Fujii]], who make optimal-policy analysis tractable by shrinking the strategy space to four enumerable profiles. Three further held frameworks now sit at different points on the same axis, which lets the claim be examined rather than repeated.

## Four Held Action Spaces

| Framework | Action space | Size | Counterfactuals available |
|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass destination — a **[[probability-surface\|surface]]** | Effectively continuous | Every destination, by construction |
| [[optimal-decisions-shot-taking-situations\|Yeung & Fujii]] | {Shoot, Pass} × {Block, Not Block} | **4 profiles** | All four, by enumeration |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | 8 directions + idle + sprint start/stop + release + pass + shot | **14 per agent** | All 14, via learned $Q$ |
| [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] | 8 directions + idle + **high pass, short pass**, shot | **12 per agent** | All 12, via learned $Q$ |

The last two are the instructive pair, and the comparison is new as of 2026-08-07.

## Two Overlapping Author Sets, Same Data, Different Action Spaces

Nakahara et al. and Fujii et al. share three of four and four of six authors respectively, use **the same 54 Meiji J1 2019 games from [[data-stadium|Data Stadium]]**, and both adapt [[google-research-football|GFootball's]] vocabulary. Their action sets still differ:

| | Nakahara et al. (14) | Fujii et al. (12) |
|---|---|---|
| Movement | 8 directions at 45° | 8 directions at 45° |
| Idle | Yes | Yes |
| Sprint state | **start, stop, release** (3 actions) | **Absent** |
| Passing | **One** pass action | **High pass, short pass** |
| Shot | Yes | Yes |

**Neither paper mentions the other's choice.** One treats sprint state as three distinct decisions and passing as one; the other the reverse.

That is not a contradiction — the two have different purposes, since a simulator must execute a pass and needs to know which kind, while an inverse valuation model reads pass type from the event stream and can ignore it. But it makes a point the field has not absorbed:

> ### `action-space-sets-the-question`
> **A framework's action space fixes which counterfactual questions it can pose, and therefore what "suboptimal" can mean within it. Frameworks with different action spaces are not measuring the same construct even when they report the same quantity.**
> ^[generated: drawn across four held sources; none states it. **Strengthened 2026-08-07** — the Nakahara/Fujii pair shows the divergence occurring within one research group on one dataset, which removes the explanation that action spaces differ because data or goals differ. rests-on: source:fernandez-pass-surface, source:yeung-four-profiles, source:nakahara-14-actions, source:fujii-12-actions]

**Nakahara et al. cannot ask "should he have played the ball long or short?"** Fujii et al. can. **Fujii et al. cannot ask "should he have started sprinting?"** Nakahara et al. can. Neither limitation is stated by either paper, and both would be invisible to a reader comparing only the reported results.

## What Each Design Buys and Costs

**Continuous surface.** Values every option at full spatial resolution. But it is defined over *destinations*, not over *what a player does* — so it can say where a pass should have gone and nothing about whether to pass at all. Exactly the "targeting within an action" limitation on [[observed-versus-optimal-decisions]].

**Radical coarsening.** Makes [[game-theory|equilibrium]] computable and the answer explainable — the payoff table *is* the explanation. The cost is that the equilibrium concerns a coarsened game, not football.

**Middle discretisation.** Small enough that every action's value can be reported, large enough to cover movement, which is where off-ball value lives. The only design that can value **a player who neither has the ball nor will receive it.**

## The Cost of the Middle, Which Neither Source States

Eight movement directions at 45° means **a run's direction is rounded to the nearest octant**. Three consequences, none stated:

1. **Two runs 40° apart may receive the same label**, and thus the same counterfactual value, though their tactical meaning differs.
2. **The action label is derived from realised velocity**, not intent. A player who intended a run and was impeded is labelled by what happened — the [[intent-vs-outcome-valuation|intent/outcome]] problem relocated from outcomes to actions.
3. **No dynamics constraints are encoded.** Every direction is treated as equally executable from every state, when a player at full sprint cannot turn 135° at will. Some "available" counterfactual actions are not available.

Nakahara et al. add a fourth: the gates that assign sprint labels (24 km/h, 0.1 m/s) are **unswept free parameters that determine label assignment** rather than value magnitude. See [[free-parameters-load-bearing]].

And a straightforward omission: the dataset contains dribbling and trapping labels, discarded "for simplicity" in Nakahara et al. and absent from Fujii et al. So the on-ball action set is narrower than [[spadl|SPADL]] and much narrower than the frameworks it is compared against.

## Size Is Not the Only Axis

This sharpens rather than contradicts the [[reinforcement-learning]] claim. Action-space size does govern *tractability* of optimal analysis. But granularity and coverage matter independently:

| | Size | Covers off-ball | Covers action *choice* | Distinguishes pass types |
|---|---|---|---|---|
| Fernández et al. | Huge | No | No | n/a — destination is the choice |
| Yeung & Fujii | Tiny | No | **Yes** | No |
| Nakahara et al. | Middling | **Yes** | **Yes** | No |
| Fujii et al. | Middling | **Yes** | **Yes** | **Yes** |

**No held framework covers fine spatial granularity, off-ball players, and choice between action types at once.** That is a gap in the field, not merely in the vault — the four papers span the design space fairly completely.

## Consequence for Cross-Framework Comparison

This supplies a candidate explanation for the vault's most striking number: **[[c-obso|C-OBSO]] and Nakahara's Q-values correlate at $\rho = 0.182$** on the same club, season and data.

C-OBSO's implicit action space is *the continuous trajectory a player ran*, valued against a predicted trajectory. Nakahara's is 14 discrete labels. Two metrics of "off-ball contribution" defined over different action spaces need not agree, and do not. See [[construct-validity]].

## The Simulator Constraint

> **Added 2026-08-07.** A dependency the vault had not recorded.

[[nfootball|NFootball's]] action set is not only a modelling choice — it is **what the simulator can execute**. A forward-approach action space is bounded below by the environment's capabilities, where an inverse one is bounded only by what the data labels.

This partly explains why Fujii et al. distinguish pass types (a simulator must decide a ball's trajectory) and drop sprint states (representing acceleration dynamics is work). **The action space of a forward framework is a statement about the simulator as much as about football**, which is one more reason results across the forward/inverse divide are not directly comparable. See [[domain-adaptation]].

## Beyond Sport

Any system valuing decisions from logs faces the same choice, usually silently: which clinical interventions count as distinct, which trades are one action, which UI events are decisions. **The discretisation determines what "a better decision" could have meant**, and it is almost never reported as a modelling assumption.

## See Also

- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[deep-q-network]]
- [[game-theory]] · [[probability-surface]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[policy-modelling]] · [[domain-adaptation]]
- [[action-valuation]] · [[off-ball-value]] · [[c-obso]] · [[spadl]] · [[intent-vs-outcome-valuation]] · [[construct-validity]]
- [[google-research-football]] · [[nfootball]] · [[data-stadium]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii]] · [[expected-value-possession-framework|EPV]]
