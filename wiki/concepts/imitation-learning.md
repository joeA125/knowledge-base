---
title: "Imitation Learning"
type: concept
tags: [imitation-learning, machine-learning, policy-modelling, markov-model, trajectory-prediction, sports-analytics, generative-model, counterfactual, reinforcement-learning, auxiliary-loss, simulator, domain-adaptation, sequence-alignment]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/expected_value_possession_framework.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 45%
  inferred: 46%
  generated: 8%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Imitation Learning

Learning a policy by **mimicking observed behaviour** rather than by optimising a reward. Given demonstrations $\{(s_t, a_t)\}$, learn $\pi(a|s)$ that reproduces them.

The simplest form — behavioural cloning — is supervised learning on state-action pairs. More elaborate forms recover a reward function from behaviour (inverse reinforcement learning) and then optimise it.

## Why Imitate Rather Than Optimise

Reinforcement learning needs a reward signal and an environment to interact with. Team sports supply neither in usable form: the reward is **sparse** (goals are rare) and intent is **unobservable**.

> **Corrected 2026-08-07, then confirmed the same day.** This page previously added "and there is no simulator faithful enough to train against". Too strong — [[google-research-football|GFootball]] exists and is well-engineered. The corrected form is that what is missing is **evidence of transfer**, not availability.
>
> That correction was made on inference. [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] now supply direct evidence: they build [[nfootball|their own simulator]], fail to reproduce demonstrated football behaviour inside it, rule out the algorithm as the cause, and attribute the failure to **"the domain-specific modeling and reality of the simulator"**. A fidelity problem, not an availability one. See [[domain-adaptation]].

[[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] frame this as a choice between two approaches to an intractable full model — estimate the components from data (the *inverse* approach) or build a simulator and generate behaviour inside it (the *forward* approach). The sports literature has overwhelmingly taken the first, and imitation learning is its core tool.

**That preference is now visibly rational rather than merely conventional.** The vault holds one forward-approach paper and it is the one that fails.

## Two Conditions That Make Demonstration the Better Option

Fujii et al. state these plainly and they generalise well past sport:

1. **Transition functions are difficult to design explicitly.**
2. **Expert demonstrations are available.**

Where both hold, learning from demonstration beats constructing an environment and doing pure RL inside it. Football satisfies both emphatically.

## The Vault's Instances

| Work | What is imitated | Purpose |
|---|---|---|
| Ghosting (Le et al., 2017) | How a league-average *defence* would move | Coaching comparison |
| Teranishi, Fujii & Takeda (2020) | Defensive movement | Defensive evaluation |
| [[trajectory-prediction\|GVRNN]] in [[c-obso]] | League-average attacking movement | **Reference for [[counterfactual-baseline]]** |
| [[policy-modelling\|Fernández et al.]] | Action and pass selection | Weighting value by likelihood |
| [[action-supervision]] in [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | Observed action choices | **Regularising a value function** |
| **[[action-supervision\|DTW-adaptive supervision]] in [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]]** | **Observed actions, realigned by state similarity** | **Regularising a value function across a domain gap** |

## Three Roles, Not One

The instances above use imitation for three structurally different purposes, and keeping them apart is what this page is for.

**1. As a policy.** The imitator's output *is* the deliverable — ghosting shows a coach how a better defence would have moved.

**2. As a measuring instrument.** The most useful idea here, and one that inverts the field's usual objective.

A learned "average behaviour" policy is normally a means to an end. In [[c-obso]] it is the instrument: train a trajectory model on opponents to learn how a typical player moves, then credit a specific player with the *difference* between what he did and what the model expected.

Two consequences follow, both awkward:

- **The objective changes.** A forecaster wants minimal error. A measuring instrument wants a well-calibrated notion of *normal*. These are not the same target, and optimising the first can degrade the second.
- **Perfection destroys the measurement.** C-OBSO is identically zero under a perfect imitator. The metric requires its own reference to be wrong.

The same tension appears in [[policy-modelling]], where the fitted behaviour policy is compared against the best available option — so a policy model that imitated *optimal* play would report nothing.

**3. As a prior on competence.** Different in kind from both.

[[action-supervision]] adds a cross-entropy loss on the softmax of learned Q-values, labelled by the action actually taken. No policy is deployed and nothing is measured against the imitator. The imitation signal exists **only to constrain the value function where data is absent** — most of a 12- or 14-action space is never visited in a given state, so those Q-values would otherwise be arbitrary.

The assumption imported is that **real agents act better than random**. Not derivable from the data, and both papers say so.

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: declared on [[action-supervision]]. **Strengthened 2026-08-07** — Fujii et al. measure the reward/imitation trade-off directly. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

Role 3 has a failure mode roles 1 and 2 do not. **Turn the weight up far enough and imitation stops being a prior and becomes the answer.** See [[imitation-reward-tradeoff]] and [[observed-versus-optimal-decisions]].

### Role 3 needs alignment when the agent acts

> **Added 2026-08-07.**

A wrinkle that only appears in the forward setting. Plain action supervision assumes the agent's timestep $t$ and the expert's timestep $t$ correspond — true when the agent never acts, false the moment it rolls out in an environment whose dynamics differ from reality, because the rollout drifts **out of phase** with the demonstration.

Fujii et al.'s fix is [[dynamic-time-warping|DTW]]: supervise against the expert action at the **most similar state**, $t' = \arg\min_j W(s,s^E)_{t,j}$, rather than the contemporaneous one.

**Nakahara et al. need none of this**, because working purely inverse means agent and demonstration share the same timesteps by construction. DTW is the price of going forward, and it is a good diagnostic for which regime a paper is in.

It also does not always work: DTW helps on a controlled mobility-gap task and **not at all** on football. Alignment corrects *phase*; it cannot repair a target environment in which the demonstrated behaviour is unavailable.

## Relation to Other Learning Regimes

| | Learns from | Optimises |
|---|---|---|
| **Imitation learning** | Demonstrations | Match to observed behaviour |
| Reinforcement learning | Reward signal | Expected return |
| Inverse RL | Demonstrations | A *reward* explaining them |
| **Imitation as auxiliary loss** | **Both** | **A weighted sum of the two** |
| Supervised learning | Labels | Match to labels |

The fourth row is where [[action-supervision]] and **DQfD** (Hester et al., 2018) sit. DQfD uses a large-margin loss forcing the expert action's $Q$ above all others; both Fujii-group papers replace it with cross-entropy, arguing that with limited data, directly maximising $Q$ at the demonstrated action beats enforcing a margin.

**Fujii et al. supply the evidence, and it is emphatic: DQfD and DQfAD score exactly zero reward on every football task**, while the cross-entropy variants reach 8.00 and 6.00. A widely used imitation loss failing outright at this data scale. See [[deep-q-network]].

The structural difference is also worth keeping: DQfD uses demonstrations to bootstrap an agent that will later act. In the inverse setting the agent never acts and the demonstrations are all there will ever be — **the supervision is not a warm start, it is permanent.**

Behavioural cloning is formally supervised learning; what distinguishes imitation learning as a field is the **sequential** setting, where the learner's own actions determine the states it later sees. That gives it the same compounding-error problem as autoregressive generation — [[teacher-forcing|exposure bias]] under a different name. The sports models sidestep it by keeping horizons short: [[trajectory-prediction|GVRNN]] predicts 4 seconds. Role 3 in the inverse setting sidesteps it entirely, since nothing is rolled out. See [[counterfactual-simulation]].

[[rlhf|RLHF]] sits between the columns — a reward model learned from human preference data, then optimised against, with a KL penalty anchoring to the base model. **The best-behaved instance of the imitation/reward trade-off in the vault**, because the coefficient is routinely reported and swept, which neither football paper does.

## Caveats

- **Demonstrations encode constraint, not just skill.** A footballer's movement reflects coaching instruction, fatigue and role as much as judgement. An imitator learns all of it undifferentiated.
- **"Average" is a population, not a person.** A league-average reference is an abstraction no individual instantiates — the same objection the vault makes of [[martingale-epv|EPVA's]] hypothetical baseline player.
- **Imitators inherit their data's biases**, including selection in who is observed doing what.
- **In role 3, the caveat becomes a conclusion.** If demonstrations encode constraint rather than judgement, then a value function regularised toward them is regularised toward constraint — and the resulting Q-values partly measure what players were *instructed* to do.
- **Imitation across a domain gap may be unachievable rather than merely imperfect.** Fujii et al.'s football agents reproduced neither the demonstration nor a reward-maximising alternative. Where the target environment cannot support the demonstrated behaviour, "how faithfully did it imitate?" has no useful answer. See [[imitation-reward-tradeoff]].

## See Also

- [[policy-modelling]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[c-obso]] · [[counterfactual-simulation]]
- [[action-supervision]] · [[domain-adaptation]] · [[dynamic-time-warping]] · [[deep-q-network]] · [[imitation-reward-tradeoff]]
- [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[reinforcement-learning]] · [[action-space-design]]
- [[teacher-forcing]] · [[generative-model]] · [[markov-game]] · [[rlhf]] · [[google-research-football]] · [[nfootball]]
- [[martingale-epv]] · [[defensive-valuation]] · [[space-creation]] · [[observed-versus-optimal-decisions]] · [[free-parameters-load-bearing]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
