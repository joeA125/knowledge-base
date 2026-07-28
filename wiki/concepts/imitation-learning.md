---
title: "Imitation Learning"
type: concept
tags: [imitation-learning, machine-learning, policy-modelling, markov-model, trajectory-prediction, sports-analytics, generative-model, counterfactual]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.8
provenance:
  extracted: 35%
  inferred: 60%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Imitation Learning

Learning a policy by **mimicking observed behaviour** rather than by optimising a reward. Given demonstrations $\{(s_t, a_t)\}$, learn $\pi(a|s)$ that reproduces them.

The simplest form — behavioural cloning — is supervised learning on state-action pairs. More elaborate forms recover a reward function from behaviour (inverse reinforcement learning) and then optimise it.

## Why Imitate Rather Than Optimise

Reinforcement learning needs a reward signal and an environment to interact with. Team sports supply neither in usable form: the reward is **sparse** (goals are rare), intent is **unobservable**, and there is no simulator faithful enough to train against.

[[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] frame this as a choice between two approaches to an intractable full model — estimate the components from data (the *inverse* approach) or build a simulator and learn inside it (the *forward* approach). The sports literature has overwhelmingly taken the first, and imitation learning is its core tool.

The key phrase from that line of work is that these models **mimic rather than optimise** the policy. That is a deliberate limitation, not a shortcoming.

## The Vault's Instances

| Work | What is imitated | Purpose |
|---|---|---|
| Ghosting (Le et al., 2017) | How a league-average *defence* would move | Coaching comparison |
| Teranishi, Fujii & Takeda (2020) | Defensive movement | Defensive evaluation |
| [[trajectory-prediction\|GVRNN]] in [[c-obso]] | League-average attacking movement | **Reference for [[counterfactual-baseline]]** |
| [[policy-modelling\|Fernández et al.]] | Action and pass selection | Weighting value by likelihood |

The last two are the interesting ones, because in both the imitator's output is **not used as a prediction**.

## Imitation as a Measuring Instrument

The most useful idea here, and one that inverts the field's usual objective.

A learned "average behaviour" policy is normally a means to an end — something to improve on, or to correct for. In [[c-obso]] it is the **measuring instrument**: train a trajectory model on opponents to learn how a typical player moves, then credit a specific player with the *difference* between what he did and what the model expected.

Two consequences follow, both awkward:

- **The objective changes.** A forecaster wants minimal error. A measuring instrument wants a well-calibrated notion of *normal*. These are not the same target, and optimising the first can degrade the second.
- **Perfection destroys the measurement.** C-OBSO is identically zero under a perfect imitator. The metric requires its own reference to be wrong.

The same tension appears in [[policy-modelling]], where the fitted behaviour policy is compared against the best available option — the gap between them is the finding, so a policy model that imitated *optimal* play would report nothing.

## Relation to Other Learning Regimes

| | Learns from | Optimises |
|---|---|---|
| **Imitation learning** | Demonstrations | Match to observed behaviour |
| Reinforcement learning | Reward signal | Expected return |
| Inverse RL | Demonstrations | A *reward* explaining them |
| Supervised learning | Labels | Match to labels |

Behavioural cloning is formally supervised learning; what distinguishes imitation learning as a field is the **sequential** setting, where the learner's own actions determine the states it later sees.

That gives it the same compounding-error problem as autoregressive generation. A model trained on expert states, then deployed on states its own errors produced, drifts into territory it never saw — which is precisely [[teacher-forcing|exposure bias]] under a different name. The sports models sidestep it by keeping horizons short: [[trajectory-prediction|GVRNN]] predicts 4 seconds, and error grows sharply beyond 8.

[[rlhf|RLHF]] sits between the columns — a reward model is learned from human preference data (inverse-RL-flavoured), then optimised against (RL).

## Caveats

- **Demonstrations encode constraint, not just skill.** A footballer's movement reflects coaching instruction, fatigue and role as much as judgement. An imitator learns all of it undifferentiated. See the caveats on [[policy-modelling]].
- **"Average" is a population, not a person.** A league-average reference is an abstraction no individual instantiates, which is the same objection the vault makes of [[martingale-epv|EPVA's]] hypothetical baseline player.
- **Imitators inherit their data's biases**, including selection in who is observed doing what.

## See Also

- [[policy-modelling]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[c-obso]]
- [[teacher-forcing]] · [[generative-model]] · [[markov-game]] · [[rlhf]]
- [[martingale-epv]] · [[defensive-valuation]] · [[space-creation]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
