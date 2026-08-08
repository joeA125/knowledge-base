---
title: "Deep Q-Network"
type: concept
tags: [reinforcement-learning, temporal-difference, deep-learning, experience-replay, training-technique, machine-learning, markov-model, multi-agent, imitation-learning]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 25%
  generated: 8%
  imported: 22%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Deep Q-Network

Approximating the action-value function $Q(s,a;\theta)$ with a neural network that emits one value per action for a given state (Mnih et al., 2015). The network is trained on the [[temporal-difference-learning|TD]] residual, but doing that naively is unstable, and most of what "DQN" names is the machinery that makes it work.

## The Three Stabilisers

| Device | Problem it addresses |
|---|---|
| **Target network** $\theta'$ | The bootstrap target moves with every update, since the same parameters produce both $Q(s_t,a_t)$ and $Q(s_{t+1}, \cdot)$. Copying $\theta$ to a frozen $\theta'$ every $\tau$ steps holds the target still |
| **Replay buffer** $\mathcal{D}_{replay}$ | Consecutive transitions are strongly correlated, violating the i.i.d. assumption gradient descent expects. Uniform resampling decorrelates them |
| **Double Q-learning** | Taking a max over noisy estimates is biased upward. DDQN selects the action with the online network and evaluates it with the target network |

The DDQN loss:

$$J_{DQ}(Q) = \sum_t^{T-1}\left(R_t + \gamma Q(s_{t+1}, a_{t+1}^{\max}; \theta') - Q(s_t, a_t; \theta)\right)^2$$

with $a_{t+1}^{\max} = \arg\max_{a_t} Q(s_{t+1}, a_t; \theta)$. **Selection and evaluation are deliberately separated**, which is the whole content of the double-Q correction.

**Prioritised experience replay** (Schaul et al., 2016) refines the buffer, sampling high-TD-error transitions more often.

## The Comparison That Makes This Page Worth Having

The vault holds two RL papers from the same group, published weeks apart, and they use opposite amounts of this machinery.

| | [[adaptive-action-supervision-multi-agent-rl\|Fujii et al. (DQAAS)]] | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |
|---|---|---|
| Algorithm | **DDQN** — off-policy | **SARSA** — on-policy |
| Target network | **Yes** | No |
| Replay buffer | **Yes**, prioritised | No |
| Double Q | **Yes** | n/a |
| Exploration | $\varepsilon$-greedy | **None — never acts** |
| Regularisation | $L_2$ | $L_1$ |
| Recurrence | No | **[[gated-recurrent-unit\|GRU]]** |
| Agent | **Acts in a simulator** | **Only observes logged data** |

The last row explains almost all the others. **The stabilisers exist to manage the feedback loop between a policy and the data it generates**, and Nakahara et al. have no such loop: their agent never acts, so the data distribution is fixed, transitions arrive in their real order, and there is nothing to decorrelate. A replay buffer over a static dataset of 1,669 possessions would be resampling, not decorrelation.

Their GRU is doing the compensating work — carrying state across the bootstrap where a target network would otherwise stabilise it. **The two papers use recurrence and stabilisers as substitutes for the same problem**, and neither notes the substitution. See [[gated-recurrent-unit]].

**The on/off-policy split follows the same logic.** SARSA's target uses the action actually taken, which is what you want when the point is to value observed football; DDQN's max is what you want when an agent must eventually choose. See [[policy-modelling]].

> ### `stabilisers-track-the-feedback-loop`
> **The DQN stabiliser set is required in proportion to how much a learning agent influences its own training distribution. Purely offline value estimation from logged behaviour needs little of it, and papers that adopt the full apparatus regardless are importing machinery for a problem they do not have.**
> ^[generated: no source states this; drawn here from the contrast between two same-group papers. rests-on: source:fujii-ddqn-full-stack, source:nakahara-sarsa-no-stabilisers]

The unexamined half of that claim is whether Nakahara et al. *should* have used a target network anyway — a moving target destabilises offline fitting too, just less. Neither paper discusses the choice, and the two were written by overlapping authors.

## DQfD: Demonstrations Inside the DQN Frame

**Deep Q-learning from Demonstrations** (Hester et al., 2018) is the direct ancestor of both vault papers' supervision schemes. Three phases: pre-train on demonstrations, sample actions in the environment, train with both losses running.

Its supervision is a **large-margin classification loss** (Piot et al., 2014):

$$J_{MS}(Q) = \sum_t \max_{a_t \in A}\left[Q(s_t,a_t) + l(a_t^E, a_t)\right] - Q(s_t, a_t^E)$$

forcing the expert action's value above every other by at least the margin $l$.

**Both Fujii-group papers replace it with cross-entropy on the softmax of $Q$**, arguing that with limited data, directly maximising $Q$ at the demonstrated action is more efficient than enforcing a margin. Fujii et al. supply the evidence: **DQfD and DQfAD score zero reward on every football task**, while the cross-entropy variants reach 8.00 and 6.00. See [[action-supervision]].

That is a clean, specific result — a widely used loss failing at the data scale this literature actually operates at — and it is the strongest methodological finding in the paper.

An important detail the buffer handles: demonstration data is **never overwritten** when the replay buffer fills. The expert transitions are permanent residents.

## Limitations Carried Into Sports Work

- **Discrete action spaces only.** DQN requires enumerable actions, which is why both papers discretise movement into 8 directions. See [[action-space-design]].
- **The stabilisers are hyperparameters.** Target update period $\tau$, buffer size, priority exponent — none reported in either held source. See [[free-parameters-load-bearing]].
- **Off-policy value estimates for unvisited actions remain extrapolation**, which is exactly why the supervision loss is needed at all.

## See Also

- [[temporal-difference-learning]] · [[action-supervision]] · [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[policy-modelling]]
- [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-learning]] · [[imitation-reward-tradeoff]] · [[action-space-design]]
- [[gated-recurrent-unit]] · [[regularization]] · [[adam-optimizer]] · [[free-parameters-load-bearing]] · [[markov-game]] · [[value-iteration]]
- [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[google-deepmind]] · [[nfootball]]
- [[adaptive-action-supervision-multi-agent-rl|Fujii et al. Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
