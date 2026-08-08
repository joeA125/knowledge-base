---
title: "Google Research Football (GFootball)"
type: entity
tags: [organisation, benchmark-environment, simulator, google, ai-research, reinforcement-learning, multi-agent, action-space, sports-analytics, evaluation, domain-adaptation]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.75
provenance:
  extracted: 52%
  inferred: 22%
  generated: 8%
  imported: 18%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Google Research Football (GFootball)

A physics-based football simulation environment for [[reinforcement-learning]] research, released by [[google-brain|Google Brain]] (Kurach et al., AAAI 2020). Agents control players in a simulated match and learn from simulated reward.

**Held only indirectly**, through two [[keisuke-fujii|Fujii-group]] sources that use it, react to it, and ultimately replace it. The environment itself is not a held source; most of what this page says about its internals is `imported:`.

## Three Uses, and They Get Progressively Less Flattering

| Source | What it takes from GFootball | What it does with it |
|---|---|---|
| Scott, Fujii & Onishi (2022) — *cited, not held* | The whole environment | Compares agent ball-passing against professional players |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | **19 actions → 14** | Applies the vocabulary to real J-League tracking; discards the dynamics |
| [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] | Pitch dimensions; action set "partially based on" it | **Rejects it and builds [[nfootball\|NFootball]]** |

Nakahara et al.'s use is the conceptually interesting one: a simulator used as a **design reference**, not as an environment. They take the ontology — what counts as an action — and leave the physics. See [[action-space-design]].

## Why It Was Rejected

> **Added 2026-08-07** on ingest of Fujii et al.

The stated reasons are specific and worth separating:

- **Transition algorithms are difficult to customise.** A tooling complaint. They needed to induce a *controlled* domain gap, which requires editing the dynamics.
- **Some commands did not work at intended timings** — passing is the named example. A **fidelity** complaint, and the serious one. A simulator in which passes do not execute when intended cannot serve as a target for imitating football, because passing is the demonstrated behaviour.

So the group's trajectory across three papers is: use it, borrow from it, replace it. See [[nfootball]] for what the replacement costs in comparability.

## The Simulator Claim, Twice Revised

[[reinforcement-learning]] once asserted that the *forward* approach — build an environment, learn a policy inside it — is unavailable in football because no simulator is faithful enough.

GFootball's existence shows the strong form is wrong: a well-engineered football simulator exists. What was missing was **evidence of transfer**, and the vault recorded that as an inference from Nakahara et al.'s borrowing pattern.

**Fujii et al. now supply the evidence directly** — and, importantly, *not* by criticising GFootball. They build their own simulator, remove GFootball's specific shortcomings, still fail to reproduce demonstrated football, and rule out the algorithm as the cause. **The fidelity problem survives the fix.**

That matters for how this page should be read. GFootball's pass-timing bug is a defect in one artefact. The gap between *any* current football simulator and real football is a property of the problem. See [[domain-adaptation]].

## Comparison to Other Shared Resources Here

| Resource | Supplies | Held | Shared? |
|---|---|---|---|
| **GFootball** | A simulator and an action vocabulary | Indirectly | **Yes** |
| [[nfootball\|NFootball]] | A simulator | Indirectly | No — one group |
| [[soccernet-game-state-reconstruction\|SoccerNet]] | Annotated broadcast video and tasks | Yes | **Yes** |
| [[camera-calibration-benchmarking\|ProCC]] | A calibration protocol and metrics | Yes | **Yes** |
| [[spadl\|SPADL]] | A common action representation | Yes | **Yes** |

GFootball and SPADL do structurally similar work from opposite directions — both impose a **shared vocabulary of what an action is**. SPADL normalises commercial event providers; GFootball defines one for simulation. Nakahara et al. chose the simulator's over the vault's incumbent, which is why their action set omits dribbles and traps.

The `Shared?` column is the one that matters for [[action-valuation-frameworks-compared|cross-framework benchmarking]]. GFootball's real value was never its physics; it was that everyone used it. **Its replacement by a bespoke environment removes a reference point without supplying another.**

## Caution

Details of GFootball beyond its action-space size, pitch dimensions and the two named defects are not established by any held source. The AAAI 2020 paper remains unheld.

## See Also

- [[nfootball]] · [[action-space-design]] · [[domain-adaptation]] · [[multi-agent-reinforcement-learning]] · [[reinforcement-learning]]
- [[action-supervision]] · [[dynamic-time-warping]] · [[deep-q-network]] · [[imitation-reward-tradeoff]] · [[counterfactual-simulation]]
- [[spadl]] · [[imitation-learning]] · [[trajectory-prediction]] · [[action-valuation-frameworks-compared]]
- [[google-brain]] · [[google-research]] · [[atom-scott]] · [[hiroshi-nakahara]] · [[keisuke-fujii]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
