---
title: "The Imitation/Reward Trade-off"
type: concept
tags: [imitation-learning, reinforcement-learning, auxiliary-loss, domain-adaptation, evaluation, model-selection, multi-agent, sports-analytics, policy-modelling, machine-learning]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.75
provenance:
  extracted: 55%
  inferred: 32%
  generated: 12%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# The Imitation/Reward Trade-off

When a model learns from demonstrations *and* from reward, two objectives compete: **reproducing what the demonstrator did**, and **obtaining reward in the target environment**. [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] state the goal as balancing "reproducibility as imitation and generalization ability to obtain rewards", note the two are "mutually independent in general", and then measure the trade-off directly.

This page exists because the vault asserted the trade-off before it had evidence for it.

## The Claim That Needed Backing

On the [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] ingest the vault declared:

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: declared on [[action-supervision]]. **Strengthened 2026-08-07** — the trade-off is now measured, though not against $\lambda_1$. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

That was drawn from Nakahara et al.'s *verbal* description of what happens at extreme $\lambda_1$: too little supervision and counterfactual values go unlearned, too much and the model overfits to observed actions. No number supported it.

**Fujii et al. supply numbers**, in a different parameterisation. The claim survives in modified form: the dial is real and measured, but $\lambda_1$ is not the only thing that turns it.

## The Measurement

Two deliberately opposed metrics: **reward obtained** (generalisation) and **[[dynamic-time-warping|DTW distance]] to the demonstration** (reproducibility — lower is better).

On chase-and-escape:

| Model | Reward @ 0.5M | DTW distance @ 0.5M |
|---|---|---|
| DQN (no demonstrations) | 0.11 | 5.70 |
| DQfD | 0.04 | 4.94 |
| DQAS | **0.26** | 4.97 |
| DQAAS | **0.29** | **4.73** |

The authors report plainly:

> The obtained rewards and DTW distance had a trade-off relationship. In general, how to strike a balance is not obvious.

And, crucially, that **the position on the frontier moves with training time**:

> with increased training steps, the DQAAS first learned the ability to maximize a reward and then learned the reproducibility at the expense of the reward

## Why the Second Quote Is the Important One

It relocates the dial. The vault had assumed the trade-off was governed by a **loss weight**; here it is governed by **how long you train**. Both are true and neither is reported alongside results in the way it would need to be for a reported gap to be interpretable.

> ### `where-you-stop-is-a-modelling-choice`
> **When imitation and reward objectives trade off during training rather than only across hyperparameter settings, the training-step count is a free parameter with the same status as a loss weight — and it is almost never reported as one.**
> ^[generated: no source frames early stopping this way; drawn here from the observed reward-then-reproducibility ordering. rests-on: source:fujii-training-step-ordering]

That belongs on [[free-parameters-load-bearing]] as a sixth kind, and it is uncomfortable because early stopping is normally treated as hygiene rather than as a substantive claim about the model. Here, stopping early gives a reward-maximising agent and stopping late gives an imitative one. **Those are different scientific claims about football, selected by a training budget.**

Note the ordering is itself surprising: reward is easier to acquire than reproducibility, and reproducibility is bought back at reward's expense. The naive expectation — that a model pre-trained on demonstrations starts imitative and drifts toward reward — is the reverse of what happened.

## The Frontier Is Not Always Reachable

The football results are the sobering half. On both football tasks DQAS and DQAAS achieve **identical reward and identical DTW distance, to zero standard error**, and the qualitative outcome is a failure on both axes at once:

- DQAAS learned to pass and shoot **without moving toward goal**
- DQN learned to move toward goal **without passing or shooting**
- The demonstration did both

So this is not a case of choosing a point on a trade-off curve. **Neither objective was met**, and the authors attribute that to simulator fidelity rather than to the balance between losses — a conclusion they support by showing that switching between decentralised and centralised MARL (DQN vs CDS) changes nothing. See [[domain-adaptation]].

**A trade-off presupposes a frontier.** Where the target environment cannot support the demonstrated behaviour, there is no frontier to sit on, and reporting a balance between two failures would be misleading. The paper avoids that trap and says so.

## Where Else This Bites

| Setting | Imitation objective | Reward objective | Trade-off reported? |
|---|---|---|---|
| [[adaptive-action-supervision-multi-agent-rl\|DQAAS]] | DTW to demonstration | Simulator reward | **Yes, measured** |
| [[action-supervision\|Nakahara et al.]] | $\mathcal{L}_{AS}$ | $\mathcal{L}_{TD}$ | Verbally, two points |
| [[rlhf\|RLHF]] | KL to the base model | Learned reward model | **Yes** — the KL coefficient |
| [[c-obso\|C-OBSO]] | Trajectory prediction accuracy | — | **Inverted**: perfect imitation zeroes the metric |
| [[policy-modelling\|Fernández et al.]] | Fitted behaviour policy | Best-available value | The gap *is* the deliverable |

[[rlhf|RLHF]] is the closest structural analogue and the best-behaved: the KL penalty anchoring a policy to its base model is exactly an imitation term weighted against a reward term, and the coefficient is routinely reported and swept. **The sports literature has the same structure and neither convention.**

The C-OBSO row is the odd one and worth keeping in view: there, better imitation *destroys* the measurement rather than trading against it. That is a third relationship — not competition, but dependence — and it means "improve the imitator" is good advice in two of these settings and bad advice in one. See [[imitation-learning]].

## What Would Settle It

1. **Report the frontier, not a point.** Sweep the supervision weight, or checkpoint across training, and plot reward against imitation distance. One curve per paper.
2. **Report the training-step count as a parameter**, with the ordering effect above stated.
3. **For valuation frameworks specifically**: plot the observed-versus-maximum Q gap against the supervision weight. That converts `optimality-gap-is-tunable` from a claim into a measurement and needs no ground truth. See [[observed-versus-optimal-decisions]].

The third remains unrun. It was the expected contribution of this paper and is not in it — see [[free-parameters-load-bearing]].

## Beyond Sport

Any system learning from logged human behaviour under a reward signal inherits this: clinical decision support anchored to physician practice, recommender systems balancing engagement against historical behaviour, autonomous driving trading comfort-matching against route efficiency. In each case the same question applies and is rarely answered — **how much of the model's apparent disagreement with humans is a finding, and how much is the weight someone chose?**

## See Also

- [[action-supervision]] · [[imitation-learning]] · [[domain-adaptation]] · [[dynamic-time-warping]] · [[deep-q-network]]
- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[policy-modelling]] · [[rlhf]] · [[c-obso]]
- [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[model-selection]] · [[construct-validity]]
- [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[nfootball]]
- [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
