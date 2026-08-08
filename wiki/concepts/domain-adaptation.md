---
title: "Domain Adaptation (Sim-to-Real and Real-to-Sim)"
type: concept
tags: [domain-adaptation, transfer-learning, reinforcement-learning, simulator, multi-agent, imitation-learning, machine-learning, sports-analytics, evaluation, animal-behaviour]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 60%
  inferred: 30%
  generated: 9%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Domain Adaptation (Sim-to-Real and Real-to-Sim)

Transferring a model across a shift between the environment it learned in (**source**) and the environment it must work in (**target**). The vault had been describing this gap for two ingests without a name for it; [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] supply one.

## The Two Directions Are Not Symmetric

| | **Sim-to-Real** | **Real-to-Sim** |
|---|---|---|
| Source | A simulator | **Real-world data** |
| Target | A physical system | A simulator |
| Source dynamics | **Known** — someone wrote them | **Unknown** — that is the whole problem |
| Typical domain | Robotics | Biological multi-agents; team sports |
| Correctable by | Randomising or tuning source parameters | **Not correctable that way** |
| Literature | Large and mature | Sparse |

Sim-to-Real is well studied because the asymmetry favours it: you control the source, so you can perturb it, randomise it, or measure the discrepancy. Real-to-Sim inverts that. **You cannot adjust the source dynamics to close the gap, because you do not know what they are** — the transition function of 22 interacting footballers is not written down anywhere.

The paper's framing makes the point precisely: on the real-world side, the transition model $\mathcal{T}^E(s'^E|s^E, \vec{a}^E)$ is *not modelled at all*, because the next state can simply be read from the data. On the simulator side a transition model $\mathcal{T}$ must be assumed. The gap between them is unmeasurable in principle.

## Why This Matters to the Vault Specifically

> **This page exists partly to consolidate a correction the vault made twice on inference and can now make on evidence.**

Until 2026-08-07 [[reinforcement-learning]] asserted that the forward approach was closed off in football because no simulator was faithful enough. That was weakened, on the [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] ingest, to:

> The forward approach is *available*; what is unavailable is evidence that a policy learned in a simulator transfers to real players.

At the time that was an inference from a behaviour — Nakahara et al. borrowing [[google-research-football|GFootball's]] action vocabulary while discarding its dynamics. It now has direct support. Fujii et al. build a simulator, fail to reproduce demonstrated football behaviour inside it, test whether the cause is algorithmic, and conclude it is not:

> the cause of the reproducibility issue may not be the centralized/decentralized or classic/recent deep RL

attributing it instead to the **"domain-specific modeling and reality of the simulator"**.

**A held source now diagnoses the football-RL bottleneck as simulator fidelity rather than algorithm choice.** The correction was right, and the vault reached it a paper early.

## Two Conditions That Make Learning From Demonstration the Better Option

The paper states these plainly and they generalise well beyond sport:

1. **Transition functions are difficult to design explicitly.**
2. **Expert demonstrations are available.**

Where both hold, learning from demonstration beats building an environment and doing pure RL inside it. Where only the first holds, you have neither route. Where only the second holds, demonstrations are a convenience rather than a necessity.

Football satisfies both emphatically — which is why the field's overwhelming preference for the **inverse** approach is a rational response to its conditions rather than a failure of ambition. See [[multi-agent-reinforcement-learning]], where the forward/inverse distinction is set out.

## The Mechanism: Adapting the Supervision, Not the Environment

Since the source dynamics cannot be corrected, the paper corrects the *supervision signal* instead.

Under a domain gap, an agent rolling out in the simulator drifts **out of phase** with the demonstration it is being supervised against — by timestep $t$ the agent is somewhere the expert reached at some other timestep. Supervising against $a_t^E$ is then supervising against the right action at the wrong moment.

The fix is [[dynamic-time-warping|DTW]]: supervise against the expert action at the **most similar state**, $t' = \arg\min_j W(s,s^E)_{t,j}$, not the contemporaneous one. See [[action-supervision]].

This generalises past RL. **Wherever a model is supervised against a reference trajectory that may run at a different rate, alignment should precede supervision** — and the usual practice of assuming index correspondence is an unstated assumption of matched dynamics.

## What the Gap Costs, Measured

The paper induces a *controlled* domain gap in the chase-and-escape task: predators move at 120% of prey speed in the source and **110% in the target**. A 10-point mobility shift, nothing else changed.

That is enough to make the task materially harder — the authors note that learning the Q-function correctly "becomes more important to catch prey" in the target. It is a useful calibration: **a domain gap does not need to be exotic to matter.** If a 10% velocity difference degrades transfer in a two-predator toy problem, the gap between real football and any simulator is not a detail to be tuned away.

DTW helps measurably on chase-and-escape and **not at all** on football (DQAS and DQAAS are identical to zero standard error on both football tasks). One reading: DTW corrects *phase* mismatch, and the football gap is not primarily a phase mismatch but a difference in what is physically possible. Alignment cannot fix a target environment in which the demonstrated behaviour is unavailable.

## Relation to Neighbouring Ideas

| | Adapts | Assumes |
|---|---|---|
| **Domain adaptation** | A model across an environment shift | Task is the same, distribution differs |
| Transfer learning | A representation across tasks | Some shared structure |
| [[imitation-learning]] | Nothing — mimics within one domain | Source and target coincide |
| [[counterfactual-simulation]] | An entity within a fixed model | The model captures the right dependencies |

The distinction from imitation learning is the one that matters here. **RoboCup imitation work has source and target environments that are basically the same**, which the paper notes explicitly. That is why robot-soccer imitation results do not transfer to this problem.

The relation to [[pre-train-then-fine-tune|transfer learning]] is a genuine family resemblance: both reuse what was learned somewhere else. The difference is that transfer learning assumes the *task* changes and the environment is whatever it is, while domain adaptation assumes the task is fixed and the *environment* changed.

## Beyond Sport

Real-to-Sim is the general shape of any attempt to build a simulator *from* observational data rather than from first principles: epidemic models fitted to case data, traffic microsimulation from loop detectors, market simulators from order books, animal collective-behaviour models from tracking. In all of them the source dynamics are unknown, the target dynamics are assumed, and **the discrepancy cannot be measured because measuring it would require the thing you lack.**

The honest response is the one this paper models: build the simulator, report what fails to reproduce, and name the fidelity gap rather than tuning until the numbers look acceptable.

## See Also

- [[dynamic-time-warping]] · [[action-supervision]] · [[imitation-reward-tradeoff]] · [[deep-q-network]]
- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[imitation-learning]] · [[pre-train-then-fine-tune]] · [[action-space-design]]
- [[nfootball]] · [[google-research-football]] · [[counterfactual-simulation]] · [[trajectory-prediction]] · [[representation-learning]]
- [[keisuke-fujii]] · [[atom-scott]] · [[naoya-takeishi]] · [[yoshinobu-kawahara]]
- [[adaptive-action-supervision-multi-agent-rl|Source Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
