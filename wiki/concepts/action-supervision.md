---
title: "Action Supervision"
type: concept
tags: [auxiliary-loss, reinforcement-learning, imitation-learning, temporal-difference, counterfactual, policy-modelling, deep-learning, sports-analytics, action-valuation, model-selection, domain-adaptation, sequence-alignment]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 68%
  inferred: 25%
  generated: 6%
  imported: 1%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-08-07
updated: 2026-08-07
---

# Action Supervision

An [[imitation-learning|imitation]] signal added as an auxiliary loss to a value-learning objective: the softmax over the learned $Q$-values is treated as a policy, and the action a real agent actually took is used as its label.

$$\mathcal{L}_{AS} = -\sum_{t \in T} \mathbf{a}_t^E \cdot \log\left(\text{softmax}(\mathbf{Q}_{s_t})\right)$$

**Both held sources are from the [[keisuke-fujii|Fujii group]] and were posted to arXiv within two weeks of each other in May 2023.**

| | [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] (2305.13030) | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] (2305.17886) |
|---|---|---|
| Role | **Proposes** the loss | **Applies** it, citing the above |
| Setting | Forward — agent acts in a simulator | Inverse — agent only observes logged data |
| Alignment | **[[dynamic-time-warping\|DTW]]-adaptive** | Contemporaneous |
| Base algorithm | DDQN + [[deep-q-network\|full stabiliser stack]] | SARSA, no stabilisers |
| Regularisation | $L_2$ | $L_1$ |
| $\lambda_1$ reported | **No** | $0.01$ |

## The Problem It Solves

[[reinforcement-learning|RL]] on logged human behaviour has a coverage problem. A [[temporal-difference-learning|TD]] update only constrains $Q(s_t, a_t)$ for the action *actually taken*. Over a 12- or 14-action space and a few thousand sequences, most state-action pairs are never visited, so their Q-values are shaped by nothing but network smoothness and the weight penalty.

Fujii et al. put it plainly: the demonstration data covers a narrow part of the state space and does not take all possible actions.

Without correction, the values of unchosen actions are close to arbitrary — and those are precisely the values a counterfactual valuation framework exists to report.

Action supervision imports an assumption to fill the gap: **real agents act better than random.** Not derivable from the data, and both papers say so.

## The Adaptive Variant

Fujii et al.'s contribution is not the loss but **which expert action it is applied against.**

Plain action supervision assumes the agent's timestep $t$ and the expert's timestep $t$ correspond. That holds in the inverse setting — Nakahara et al. never roll out, so agent and demonstration share the same timesteps by construction. It fails the moment an agent acts in a simulator whose dynamics differ from reality: the rollout drifts **out of phase** with the demonstration.

$$t' = \underset{j}{\arg\min}\, W(s, s^E)_{t,j}, \qquad \mathcal{L}_{AS+DA} = -\sum_t \mathbf{a}_{t'}^E \cdot \log\left(\text{softmax}(\mathbf{q}_{s_t})\right)$$

Supervise against the expert action at the **most similar state**, not the contemporaneous one. See [[dynamic-time-warping]] and [[domain-adaptation]].

**DTW is the price of going forward.** That is the cleanest way to state the difference between the two papers.

### And it barely helps in football

| Task | Domain gap | DTW variant better? |
|---|---|---|
| Chase-and-escape | Mobility 120% → 110% | **Yes** |
| Football 2v2, 4v8 | Real players → simulator | **No** — DQAS and DQAAS identical, zero standard error |

DTW corrects *phase* mismatch. The football gap is not a phase mismatch, so alignment has nothing to fix. See [[dynamic-time-warping]].

## Why Cross-Entropy Rather Than a Margin

The ancestor is **DQfD** (Hester et al., 2018), which uses a large-margin classification loss (Piot et al., 2014) forcing the expert action's value above all others by at least $l$:

$$J_{MS}(Q) = \sum_t \max_{a_t \in A}\left[Q(s_t,a_t) + l(a_t^E, a_t)\right] - Q(s_t, a_t^E)$$

Both Fujii-group papers replace it, arguing that **with limited data, directly maximising $Q$ at the demonstrated action is more efficient than enforcing a margin.**

Fujii et al. supply the evidence, and it is emphatic: **DQfD and DQfAD score exactly zero reward on every football task**, while the cross-entropy variants reach 8.00 (2v2) and 6.00 (4v8). A widely used loss failing outright at this data scale is the paper's strongest methodological result. See [[deep-q-network]].

The difference between DQfD and this method is also structural, not only in the loss. DQfD uses demonstrations to bootstrap an agent that will later act. In the inverse setting the agent never acts and the demonstrations are all there will ever be — **the supervision is not a warm start, it is permanent.**

## The Prediction This Page Made, and Lost

> ⚠️ **Corrected 2026-08-07.** This page previously recorded arXiv:2305.13030 as cited-not-held and stated it "would presumably answer the open $\lambda_1$ question". It has now been acquired. **It does not answer it.**

What the dedicated methodological paper on action supervision actually reports about its own supervision weight:

- **No value for $\lambda_1$ or $\lambda_2$ anywhere.** The text says only that "the $\lambda$ parameters control the weight of these losses."
- An ablation over **which losses are present** (DQN / DQfD / DQfAD / DQAS / DQAAS), not over their weights.
- No sweep, no curve.

So the paper that proposes the loss is **less specific about its weight than the applied paper that borrows it.** Nakahara et al. at least state $\lambda_1 = 0.01$ and compare against $0$.

The $\lambda_1$ question is unchanged and is now harder, because the obvious place to look has been checked. See [[free-parameters-load-bearing]].

## The Trade-off Is Real, and Now Measured

The vault declared `optimality-gap-is-tunable` on Nakahara et al.'s *verbal* description of the extremes. Fujii et al. supply numbers — in a different parameterisation.

| $\lambda_1$ | Consequence (Nakahara et al., stated not measured) |
|---|---|
| $\ll 0.01$ | Insufficient learning of counterfactual action values |
| $\approx 0.01$ | The chosen setting |
| $\gg 0.01$ | **Overfits to actual actions**; stops distinguishing counterfactuals |

Fujii et al. measure the same tension between **reward obtained** and **[[dynamic-time-warping|DTW distance]] to the demonstration**, report that the two "had a trade-off relationship", and observe that the position on the frontier moves with **training time** — reward is learned first, reproducibility afterwards "at the expense of the reward".

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: **strengthened 2026-08-07** — the trade-off is now measured, though against training steps and method choice rather than against $\lambda_1$. The claim survives in modified form: the dial is real; $\lambda_1$ is not the only knob. Also on [[observed-versus-optimal-decisions]], [[reinforcement-learning]], [[free-parameters-load-bearing]]. rests-on: source:nakahara-lambda-tradeoff, source:fujii-reward-dtw-tradeoff]

See [[imitation-reward-tradeoff]], which now carries the full argument.

## Why This Parameter Is Different From the Others

$\lambda_1$ does not bound a horizon, shape a decay, or gate an event. It sets **how strongly the model's notion of a good action is pulled toward what humans did.**

| $\lambda_1 \to 0$ | $\lambda_1 \to \infty$ |
|---|---|
| Pure value learning | Pure [[imitation-learning\|imitation]] |
| Optimal-policy flavoured | [[policy-modelling\|Policy modelling]] |
| Unconstrained counterfactuals | No counterfactuals |
| Players look arbitrary | **Players look optimal by construction** |

Every other free parameter catalogued in this vault, if wrong, produces wrong *values* or the wrong *event set*. A wrong $\lambda_1$ produces a wrong **conclusion about football**. See [[observed-versus-optimal-decisions]].

## The Odd Empirical Result

In Nakahara et al., adding supervision **halved the TD loss** (0.0034 against 0.0063) but **slightly worsened the supervision loss itself** (3.9550 against 3.9407).

Neither paper comments. Two readings remain undistinguished: the losses genuinely conflict and the weighted sum trades a little $\mathcal{L}_{AS}$ for a lot of $\mathcal{L}_{TD}$; or 3.94 sits near the entropy floor of a 14-action distribution ($\ln 14 \approx 2.64$, so both models are worse than uniform) and neither is learning the policy at all.

**Fujii et al. do not resolve this either**, since they report no loss values — only reward and DTW distance. The second reading would substantially weaken Nakahara et al.'s qualitative argument and nothing yet rules it out.

## Beyond Sport

The pattern applies wherever a value function must be learned from logged behaviour that cannot be re-run: clinical treatment policies, recommendation logs, industrial control traces. In each case the counterfactual arm is unvisited by construction, and in each case the fix is to import a prior about the quality of the logged behaviour.

Two things follow from this ingest. **The strength of that prior is a free parameter and is rarely reported** — even by the paper that introduces it. And **where the logged behaviour comes from a different environment than the one being modelled, the prior must be aligned before it is applied**, which almost nobody does.

## See Also

- [[dynamic-time-warping]] · [[domain-adaptation]] · [[deep-q-network]] · [[imitation-reward-tradeoff]] · [[temporal-difference-learning]]
- [[multi-agent-reinforcement-learning]] · [[action-space-design]] · [[reinforcement-learning]] · [[imitation-learning]] · [[policy-modelling]]
- [[regularization]] · [[multi-task-learning]] · [[model-selection]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[rlhf]]
- [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[nfootball]]
- [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[kazushi-tsutsui]] · [[atom-scott]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]]
