---
title: "Imitation Learning"
type: concept
tags: [imitation-learning, machine-learning, policy-modelling, markov-model, trajectory-prediction, sports-analytics, generative-model, counterfactual, reinforcement-learning, auxiliary-loss, simulator]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/expected_value_possession_framework.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 36%
  inferred: 55%
  generated: 7%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Imitation Learning

Learning a policy by **mimicking observed behaviour** rather than by optimising a reward. Given demonstrations $\{(s_t, a_t)\}$, learn $\pi(a|s)$ that reproduces them.

The simplest form — behavioural cloning — is supervised learning on state-action pairs. More elaborate forms recover a reward function from behaviour (inverse reinforcement learning) and then optimise it.

## Why Imitate Rather Than Optimise

Reinforcement learning needs a reward signal and an environment to interact with. Team sports supply neither in usable form: the reward is **sparse** (goals are rare) and intent is **unobservable**.

> **Corrected 2026-08-07.** This page previously added "and there is no simulator faithful enough to train against". Too strong — [[google-research-football|GFootball]] exists and is well-engineered. What is missing is **evidence of transfer**: a policy optimised against a simulator's physics is optimal for that simulator, and nothing establishes it says anything about real players. A fidelity problem, not an availability one. The same correction was made on [[reinforcement-learning]].

[[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] frame this as a choice between two approaches to an intractable full model — estimate the components from data (the *inverse* approach) or build a simulator and learn inside it (the *forward* approach). The sports literature has overwhelmingly taken the first, and imitation learning is its core tool.

[[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] restate the same distinction and then behave in a way that illustrates the corrected claim precisely: they borrow GFootball's **action vocabulary** and discard its **dynamics**, learning entirely from logged J-League tracking. Researchers who trust a simulator's ontology but not its physics take the action set and leave the rest. See [[action-space-design]].

The key phrase from that line of work is that these models **mimic rather than optimise** the policy. That is a deliberate limitation, not a shortcoming.

## The Vault's Instances

| Work | What is imitated | Purpose |
|---|---|---|
| Ghosting (Le et al., 2017) | How a league-average *defence* would move | Coaching comparison |
| Teranishi, Fujii & Takeda (2020) | Defensive movement | Defensive evaluation |
| [[trajectory-prediction\|GVRNN]] in [[c-obso]] | League-average attacking movement | **Reference for [[counterfactual-baseline]]** |
| [[policy-modelling\|Fernández et al.]] | Action and pass selection | Weighting value by likelihood |
| **[[action-supervision]]** in [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | Observed action choices | **Regularising a value function** |

## Three Roles, Not One

The instances above use imitation for three structurally different purposes, and keeping them apart is what this page is for.

**1. As a policy.** The imitator's output *is* the deliverable — ghosting shows a coach how a better defence would have moved.

**2. As a measuring instrument.** The most useful idea here, and one that inverts the field's usual objective.

A learned "average behaviour" policy is normally a means to an end. In [[c-obso]] it is the instrument: train a trajectory model on opponents to learn how a typical player moves, then credit a specific player with the *difference* between what he did and what the model expected.

Two consequences follow, both awkward:

- **The objective changes.** A forecaster wants minimal error. A measuring instrument wants a well-calibrated notion of *normal*. These are not the same target, and optimising the first can degrade the second.
- **Perfection destroys the measurement.** C-OBSO is identically zero under a perfect imitator. The metric requires its own reference to be wrong.

The same tension appears in [[policy-modelling]], where the fitted behaviour policy is compared against the best available option — so a policy model that imitated *optimal* play would report nothing.

**3. As a prior on competence.** New on 2026-08-07, and different in kind from both.

[[action-supervision]] adds a cross-entropy loss on the softmax of learned Q-values, labelled by the action actually taken, weighted by $\lambda_1$. No policy is deployed and nothing is measured against the imitator. The imitation signal exists **only to constrain the value function where data is absent** — most of a 14-action space is never visited in any given state, so those Q-values would otherwise be arbitrary.

The assumption imported is that **real players act better than random**. That is not derivable from the data, and the authors say so.

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff]

Role 3 has a failure mode roles 1 and 2 do not. **Turn the weight up far enough and imitation stops being a prior and becomes the answer** — the model reproduces observed choices and reports no suboptimality at all. The source states this outright and does not measure it. See [[observed-versus-optimal-decisions]].

## Relation to Other Learning Regimes

| | Learns from | Optimises |
|---|---|---|
| **Imitation learning** | Demonstrations | Match to observed behaviour |
| Reinforcement learning | Reward signal | Expected return |
| Inverse RL | Demonstrations | A *reward* explaining them |
| **Imitation as auxiliary loss** | **Both** | **A weighted sum of the two** |
| Supervised learning | Labels | Match to labels |

The fourth row is where [[action-supervision]] and DQfD (Hester et al., 2018) sit — nearest neighbour to Nakahara et al.'s method and cited by it. The difference is that DQfD uses demonstrations to bootstrap an agent that will later act, whereas here the agent never acts and the demonstrations are all there will ever be. **The supervision is not a warm start; it is permanent.**

Behavioural cloning is formally supervised learning; what distinguishes imitation learning as a field is the **sequential** setting, where the learner's own actions determine the states it later sees.

That gives it the same compounding-error problem as autoregressive generation — precisely [[teacher-forcing|exposure bias]] under a different name. The sports models sidestep it by keeping horizons short: [[trajectory-prediction|GVRNN]] predicts 4 seconds, and error grows sharply beyond 8. Role 3 sidesteps it entirely, since nothing is rolled out. See [[counterfactual-simulation]], where the same trade is set out as re-generation against value-function readout.

[[rlhf|RLHF]] sits between the columns — a reward model is learned from human preference data (inverse-RL-flavoured), then optimised against (RL).

## Caveats

- **Demonstrations encode constraint, not just skill.** A footballer's movement reflects coaching instruction, fatigue and role as much as judgement. An imitator learns all of it undifferentiated. See the caveats on [[policy-modelling]].
- **"Average" is a population, not a person.** A league-average reference is an abstraction no individual instantiates, which is the same objection the vault makes of [[martingale-epv|EPVA's]] hypothetical baseline player.
- **Imitators inherit their data's biases**, including selection in who is observed doing what.
- **In role 3, the caveat becomes a conclusion.** If demonstrations encode constraint rather than judgement, then a value function regularised toward them is regularised toward constraint — and the resulting Q-values partly measure what players were *instructed* to do.

## See Also

- [[policy-modelling]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[c-obso]] · [[counterfactual-simulation]]
- [[action-supervision]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[reinforcement-learning]] · [[action-space-design]]
- [[teacher-forcing]] · [[generative-model]] · [[markov-game]] · [[rlhf]] · [[google-research-football]]
- [[martingale-epv]] · [[defensive-valuation]] · [[space-creation]] · [[observed-versus-optimal-decisions]] · [[free-parameters-load-bearing]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
