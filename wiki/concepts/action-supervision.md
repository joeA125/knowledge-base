---
title: "Action Supervision"
type: concept
tags: [auxiliary-loss, reinforcement-learning, imitation-learning, temporal-difference, counterfactual, policy-modelling, deep-learning, sports-analytics, action-valuation, model-selection]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.75
provenance:
  extracted: 62%
  inferred: 30%
  generated: 7%
  imported: 1%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Action Supervision

An [[imitation-learning|imitation]] signal added as an auxiliary loss to a value-learning objective: the softmax over the learned $Q$-values is treated as a policy, and the action a real agent actually took is used as its label.

$$\mathcal{L}_{AS} = -\sum_{t \in T} \mathbf{a}_t \cdot \log\left(\text{softmax}(\mathbf{Q}_{s_t})\right)$$

Introduced to this vault by [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]], who combine it with a [[temporal-difference-learning|TD]] loss and $L_1$ [[regularization|regularisation]]:

$$\mathcal{L}_{total} = \mathcal{L}_{TD} + \lambda_1 \mathcal{L}_{AS} + \lambda_2 \mathcal{L}_{L_1}$$

## The Problem It Solves

[[reinforcement-learning|RL]] on logged human behaviour has a coverage problem. A [[temporal-difference-learning|TD]] update only constrains $Q(s_t, a_t)$ for the action *actually taken*. Over a 14-action space and a few thousand possessions, most state-action pairs are never visited, so their Q-values are shaped by nothing but network smoothness and the $L_1$ penalty.

Without correction, the values of unchosen actions are close to arbitrary — and those are precisely the values a counterfactual valuation framework exists to report.

Action supervision imports an assumption to fill the gap: **real agents act better than random.** That is not derivable from the data, and the authors say so — it is an inductive bias, not a finding.

## The Trade-off Is the Whole Point

The supervision weight $\lambda_1$ controls a genuine tension, and the source states both ends:

| $\lambda_1$ | Consequence |
|---|---|
| $\ll 0.01$ | Insufficient learning of counterfactual action values — unchosen actions stay unconstrained |
| $\approx 0.01$ | The chosen setting |
| $\gg 0.01$ | **Overfits to actual actions**; the model stops distinguishing counterfactuals |

This is unusually candid. Most free parameters in this vault are asserted with a one-line justification and no discussion of what varying them would do — see [[free-parameters-load-bearing]]. Here the authors describe the failure mode at each extreme precisely, and then **do not measure it**, comparing only $\lambda_1 = 0$ against $\lambda_1 = 0.01$.

## Why This Parameter Is Different From the Others

Every other free parameter catalogued in this vault is a **horizon**, a **shape**, or a **gate**. $\lambda_1$ is none of those. It sets **how strongly the model's notion of a good action is pulled toward what humans did.**

That makes it the first parameter here that directly interpolates between two of the vault's own categories:

| $\lambda_1 \to 0$ | $\lambda_1 \to \infty$ |
|---|---|
| Pure value learning | Pure [[imitation-learning\|imitation]] |
| Optimal-policy flavoured | [[policy-modelling\|Policy modelling]] |
| Unconstrained counterfactuals | No counterfactuals |

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: no source states this; drawn here from the source's own description of the $\lambda_1$ extremes. Also on [[observed-versus-optimal-decisions]] and [[reinforcement-learning]]. rests-on: source:nakahara-lambda-tradeoff]

That claim matters beyond this paper. [[observed-versus-optimal-decisions]] asks whether players really decide badly or whether models only think so. Action supervision supplies a third possibility: **the size of the gap is a modelling choice.** Turn $\lambda_1$ up and players look wise; turn it down and they look foolish. Nobody has reported the curve.

## The Odd Empirical Result

Adding supervision **halved the TD loss** (0.0034 against 0.0063) but **slightly worsened the supervision loss itself** (3.9550 against 3.9407). The auxiliary objective got worse when it was added to the objective.

The source does not comment on this. Two readings are available and it does not distinguish them: either the two losses genuinely conflict, and the weighted sum trades a little $\mathcal{L}_{AS}$ for a lot of $\mathcal{L}_{TD}$; or 3.94 is near the entropy floor of a 14-action distribution ($\ln 14 \approx 2.64$, so the model is worse than uniform on both counts) and neither model is learning the policy at all. **The second possibility would substantially weaken the paper's qualitative argument**, and no reported number rules it out.

## Relation to Neighbouring Ideas

| Approach | What supplies the extra signal | Where it enters |
|---|---|---|
| **Action supervision** | Observed actions as labels on softmax-$Q$ | Auxiliary loss on the value function |
| [[imitation-learning\|Behavioural cloning]] | Observed actions as labels | *The whole objective* |
| DQfD (Hester et al. 2018) | Expert demonstrations | Margin loss forcing expert action's $Q$ above others |
| [[rlhf\|RLHF]] | Human preferences | A *learned reward*, then policy optimisation |
| [[multi-task-learning]] | A related task's labels | Shared representation |

Nearest neighbour is **Deep Q-learning from Demonstrations**, which the source cites directly. The difference is that DQfD uses demonstrations to bootstrap an agent that will later act; here the agent never acts, and the demonstrations are all there will ever be. The supervision is not a warm start — it is permanent.

Also cited, not held: **Fujii, Tsutsui, Scott, Nakahara et al. (2023)**, *Adaptive action supervision in reinforcement learning from real-world multi-agent demonstrations*, arXiv:2305.13030 — the same group's dedicated treatment, which would presumably answer the $\lambda_1$ question.

## Beyond Sport

The pattern applies wherever a value function must be learned from logged behaviour that cannot be re-run: clinical treatment policies, recommendation logs, industrial control traces. In each case the counterfactual arm is unvisited by construction, and in each case the fix is to import a prior about the quality of the logged behaviour. **The strength of that prior is a free parameter, and it is rarely reported.**

## See Also

- [[temporal-difference-learning]] · [[multi-agent-reinforcement-learning]] · [[action-space-design]] · [[reinforcement-learning]]
- [[imitation-learning]] · [[policy-modelling]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[rlhf]]
- [[regularization]] · [[multi-task-learning]] · [[model-selection]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]]
- [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[kazushi-tsutsui]]
- [[action-valuation-multi-agent-reinforcement-learning|Source Summary]]
