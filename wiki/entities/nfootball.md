---
title: "NFootball"
type: entity
tags: [organisation, benchmark-environment, simulator, ai-research, reinforcement-learning, multi-agent, action-space, sports-analytics, evaluation, domain-adaptation, single-source]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.7
provenance:
  extracted: 62%
  inferred: 24%
  generated: 13%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# NFootball

A football RL environment written by the [[nagoya-university|Nagoya]] group for [[adaptive-action-supervision-multi-agent-rl|Fujii et al. (2023)]], and released with their code. The vault's second simulator after [[google-research-football|GFootball]].

## Why It Exists

The authors state the reason directly: in GFootball, the **transition algorithms are difficult to customise** and some commands — passing is the example given — **did not work at their intended timings**.

So NFootball is a *reaction* to GFootball rather than an independent effort, and the two complaints are worth separating:

- **Customisability** is a research-tooling complaint. They needed to induce a controlled domain gap, which means editing the dynamics.
- **Pass timing** is a fidelity complaint, and the more interesting one. A simulator in which passes do not execute when intended cannot serve as a target for imitating football, because passing *is* the demonstrated behaviour.

## Specification

| | |
|---|---|
| Pitch | $x \in [-1, 1]$, $y \in [-0.42, 0.42]$ — **copied from GFootball** |
| Goal | $y \in [-0.044, 0.044]$ |
| Timestep | 0.1 s; episode limit 8.5 s |
| Actions | **12** — 8 movement directions at 45° (relative coordinates), idle, high pass, short pass, shot — "partially based on GFootball" |
| Tasks held | 2v2, 4v8 |
| Implementation | Python throughout, "and then transparent" |

Continuous space, discrete time, two dimensions — the same shape as the MAPE predator-prey environment the group also uses.

Note the action space is **12 here against 14 in [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]]**, from overlapping authors on the same J-League data. NFootball splits passing into high and short and drops the sprint-state actions; Nakahara et al. keep sprint start/stop/release and have a single pass. Neither paper mentions the other's choice. See [[action-space-design]].

## The Cost of Building Your Own

The paper does not raise this, and it is the main reason this page exists rather than a line on the GFootball page.

**A bespoke simulator means no one can benchmark against you.** GFootball's value is not its physics but that it is *shared* — Scott, Fujii & Onishi (2022) could compare RL agents against real football strategies partly because GFootball is a fixed reference point. Results in NFootball are comparable only to other NFootball results, of which there is currently one paper's worth.

That is the [[action-valuation-frameworks-compared|`no-cross-framework-benchmarking`]] problem appearing in a new place. Where the vault has previously observed that groups do not benchmark against competitors' *metrics*, here a group has moved the *environment* out from under any possible comparison — for defensible reasons, and with the same effect.

> ### `bespoke-environments-foreclose-comparison`
> **A research group that builds its own simulator to fix a shared one's shortcomings trades external comparability for internal control, and the trade is rarely acknowledged as a cost.**
> ^[generated: no source raises this; drawn here from the paper's stated motivation and the absence of any GFootball comparison. rests-on: source:fujii-nfootball-motivation, absence:no-held-source-benchmarks-across-frameworks]

The mitigation the authors do apply is releasing the code, which makes the environment reproducible even if it is not yet a reference point. That is more than most.

## What Its Failure Establishes

The most valuable thing in the vault about NFootball is that **it did not work well enough**.

Neither DQAAS nor the baselines reproduced demonstrated football behaviour inside it. The authors test whether the cause is algorithmic — decentralised against centralised MARL, classic against recent — conclude it is not, and attribute the failure to "the domain-specific modeling and reality of the simulator".

**A purpose-built simulator, built by domain experts specifically to fix the incumbent's problems, still could not support imitation of real football.** That is stronger evidence for the fidelity gap than any argument from GFootball's shortcomings, because it removes the obvious objection that a better-suited environment would have worked. See [[domain-adaptation]] and [[reinforcement-learning]].

## Caution

Everything here comes from one paper's methods section. Whether NFootball has been used since, by this group or others, is not established.

## See Also

- [[google-research-football]] · [[domain-adaptation]] · [[action-space-design]] · [[imitation-reward-tradeoff]]
- [[multi-agent-reinforcement-learning]] · [[reinforcement-learning]] · [[deep-q-network]] · [[dynamic-time-warping]] · [[counterfactual-simulation]]
- [[spadl]] · [[camera-calibration-benchmarking]] · [[soccernet-game-state-reconstruction]] · [[action-valuation-frameworks-compared]]
- [[keisuke-fujii]] · [[atom-scott]] · [[nagoya-university]] · [[data-stadium]]
- [[adaptive-action-supervision-multi-agent-rl|Source Summary]]
