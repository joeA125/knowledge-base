---
title: "Google Research Football (GFootball)"
type: entity
tags: [organisation, benchmark-environment, simulator, google, ai-research, reinforcement-learning, multi-agent, action-space, sports-analytics, evaluation, single-source]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.65
provenance:
  extracted: 40%
  inferred: 25%
  generated: 5%
  imported: 30%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Google Research Football (GFootball)

A physics-based football simulation environment for [[reinforcement-learning]] research, released by [[google-brain|Google Brain]] (Kurach et al., AAAI 2020). Agents control players in a simulated match and learn from simulated reward.

**Held only indirectly**, via [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]], who borrow its action vocabulary. The environment itself is not a held source, and most of what this page says about its internals is `imported:` — the source describes only the action set.

## Why It Has a Page

Because of what Nakahara et al. did with it, which is more interesting than using it.

They took GFootball's **19 discrete actions**, reduced them to **14** for attacking players, and applied that vocabulary to **real J-League tracking data** — while using none of GFootball's dynamics, none of its rewards, and none of its simulated experience. See [[action-space-design]].

That is a simulator used as a **design reference**, not as an environment.

## The Simulator Claim, Qualified

[[reinforcement-learning]] carries the assertion that the *forward* approach — build an environment, learn a policy inside it — is unavailable in football because there is **no simulator faithful enough to interact with**.

GFootball shows the claim needs restating. Simulators exist, and are well-engineered. What is missing is **transfer**: a policy optimised against GFootball's physics is optimal for GFootball, and nothing establishes that it is informative about Yokohama F. Marinos. Scott, Fujii & Onishi (2022) — cited by Nakahara et al., not held — apparently address exactly this question, comparing RL and real-world football strategies.

So the correct form is:

> The forward approach is available in football; what is unavailable is evidence that behaviour learned in a simulator transfers to real players.

That is a weaker and more accurate claim, and it is a *fidelity* problem rather than an *availability* one. [[multi-agent-reinforcement-learning]] sets out the forward/inverse distinction this turns on.

**The borrowing is the tell.** Nakahara et al. take the action vocabulary and discard the dynamics — which is precisely the behaviour of researchers who trust a simulator's *ontology* but not its *physics*.

## Comparison to Other Shared Resources Here

| Resource | Supplies | Held |
|---|---|---|
| **GFootball** | A simulator and an action vocabulary | Indirectly |
| [[soccernet-game-state-reconstruction\|SoccerNet]] | Annotated broadcast video and tasks | Yes |
| [[camera-calibration-benchmarking\|ProCC]] | A calibration protocol and metrics | Yes |
| [[spadl\|SPADL]] | A common action representation | Yes |

GFootball and SPADL are doing structurally similar work — both impose a **shared vocabulary of what an action is** — from opposite directions. SPADL normalises the action sets of commercial event providers; GFootball defines one for simulation. Nakahara et al. chose the simulator's over the vault's incumbent, and the consequence is that their action set omits dribbles and traps, which SPADL includes.

## Caution

Details of GFootball beyond its action-space size are not established by any held source. The AAAI 2020 paper is a natural acquisition target if the RL line is pursued further.

## See Also

- [[action-space-design]] · [[multi-agent-reinforcement-learning]] · [[reinforcement-learning]] · [[action-supervision]]
- [[spadl]] · [[imitation-learning]] · [[counterfactual-simulation]] · [[trajectory-prediction]]
- [[google-brain]] · [[google-research]] · [[hiroshi-nakahara]] · [[keisuke-fujii]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
