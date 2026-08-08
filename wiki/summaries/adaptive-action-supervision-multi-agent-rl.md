---
title: "Adaptive Action Supervision in Reinforcement Learning from Real-World Multi-Agent Demonstrations"
type: source_summary
tags: [summary, reinforcement-learning, multi-agent, domain-adaptation, simulator, imitation-learning, auxiliary-loss, sequence-alignment, temporal-difference, experience-replay, action-space, animal-behaviour, sports-analytics, optical-tracking-data, evaluation]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 76%
  inferred: 18%
  generated: 5%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-08-07
updated: 2026-08-07
---

# Adaptive Action Supervision in RL from Real-World Multi-Agent Demonstrations

**Fujii, Tsutsui, Scott, Nakahara, Takeishi & Kawahara** — [[keisuke-fujii]] (**first author**, unusually), [[kazushi-tsutsui]], [[atom-scott]], [[hiroshi-nakahara]], [[naoya-takeishi]], [[yoshinobu-kawahara]]. [[nagoya-university]], RIKEN AIP, JST PRESTO, [[university-of-tokyo]], [[osaka-university]]. arXiv:2305.13030, ICAART.

This was flagged on 2026-08-07 as the **highest-value acquisition target** arising from the [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] ingest, on the expectation that it would settle the open $\lambda_1$ question recorded on [[action-supervision]], [[free-parameters-load-bearing]] and [[observed-versus-optimal-decisions]].

> ⚠️ **It does not.** See "The Prediction That Failed" below. The vault's own forecast about a source was wrong, and three pages have been corrected rather than resolved.

## The Problem: Real-to-Sim

The framing is the paper's most portable contribution and the vault had no name for it.

**Sim-to-Real** (robotics, well studied) transfers a policy learned in a simulator to a real robot. The source dynamics — the simulator's — are **known**, because someone wrote them.

**Real-to-Sim** is the inversion this paper names: the source is *real-world data with unknown governing equations*, and the target is a simulator built to approximate it. You cannot correct for a domain gap by adjusting the source, because you do not know what the source dynamics are.

Football qualifies twice over: the transition function of 22 interacting players is not writable down, and expert demonstrations (professional tracking data) are abundant. The paper argues that when both conditions hold, learning from demonstration beats pure RL. See [[domain-adaptation]].

## Method: DQAAS

Built on **DQfD** (Hester et al., 2018), in three phases: pre-train on demonstrations → sample actions in the target environment → train in the environment. See [[deep-q-network]] for the machinery.

Total loss:

$$J(Q) = J_{DQ}(Q) + \lambda_1 J_{AS+DA}(Q) + \lambda_2 J_{L_2}(Q)$$

### Two changes to DQfD

**1. Cross-entropy instead of large-margin.** DQfD uses a margin loss forcing the expert action's $Q$ above all others by at least $l$:

$$J_{MS}(Q) = \sum_t \max_{a_t} [Q(s_t,a_t) + l(a_t^E, a_t)] - Q(s_t, a_t^E)$$

The authors replace it with cross-entropy on the softmax of $Q$ — **the exact loss [[action-supervision|Nakahara et al. use]]** — arguing that with limited data, directly maximising $Q$ at the demonstrated action is more efficient than enforcing a margin.

**2. Dynamic time warping to choose *which* expert action to supervise against.** The novel part, and the reason for "adaptive".

Both DQfD's margin loss and plain action supervision assume **the expert's timestep $t$ and the agent's timestep $t$ correspond.** Under a domain gap they do not — the agent's rollout drifts out of phase with the demonstration. So:

$$t' = \arg\min_j W(s, s^E)_{t,j}, \qquad J_{AS+DA}(Q_t) = -\sum_t \mathbf{a}_{t'}^E \cdot \log(\text{softmax}(\mathbf{q}_{s_t}))$$

where $W$ is a [[dynamic-time-warping|DTW]] warping-path matrix over state sequences. **Supervise against the expert action at the most similar state, not the contemporaneous one.**

This cleanly explains why Nakahara et al. need no DTW: they never roll out. Working purely inverse on real data, agent and demonstration timesteps are the same timesteps by construction. **DTW is the price of going forward.**

## Experiments

Two tasks, both with a *deliberately induced* domain gap.

| Task | Agents | Actions | Gap induced by |
|---|---|---|---|
| Chase-and-escape (MAPE) | 2 predators, 1 prey | 13 (12 directions at 30° + idle) | Predator mobility 120% of prey in source, **110% in target** |
| Football 2v2 and 4v8 (NFootball) | 2 or 4 attackers | 12 (8 directions at 45°, idle, high pass, short pass, shot) | Real players vs. a simulator |

Baselines: DQN (no demonstrations), DQfD, DQfAD (DQfD + DTW), DQAS (action supervision, no DTW), DQAAS (full). Plus **CDS** (Li et al., 2021), a centralised MARL method, on 4v8.

Two metrics, deliberately opposed: **reward obtained** (generalisation) and **[[dynamic-time-warping|DTW distance]] to the demonstration** (reproducibility as imitation).

### Football data

The **same 54 games of Meiji J1 League 2019 from [[data-stadium|Data Stadium]]** as Nakahara et al. and [[c-obso|C-OBSO]] — the fourth held source on this dataset. 198 last-pass-and-goal sequences and 1,385 last-pass sequences; 1,121/125/139 split for pre-training. Target RL used **16 training and 5 test episodes**.

### Results

| Task | Best reward | Best DTW distance |
|---|---|---|
| Chase-and-escape | DQAAS 0.29 ± 0.09 | DQAAS 4.73 ± 1.07 (DQN pre-trained better at 4.12) |
| Football 2v2 | DQAS = DQAAS 8.00 ± 0.00 | DQAS = DQAAS 2.25 ± 0.00 |
| Football 4v8 | DQAS = DQAAS = CDS+AS/AAS 6.00 | DQN 3.24, CDS 3.25 — **baselines competitive** |

**Action supervision is what matters; DTW adds little in football.** DQAS and DQAAS are identical on both football tasks, to zero standard error. DTW helps measurably only on chase-and-escape, where the domain gap is a single tunable mobility ratio.

**DQfD and DQfAD fail outright** — zero reward on every football task. The margin loss does not work at this data scale, which is the paper's substantive case for the cross-entropy substitution.

**Zero standard error across five seeds** on both football tasks is worth flagging. Identical performance from five random initialisations suggests convergence to one degenerate solution, not robustness.

## The Honest Negative Result

The authors report a failure clearly, and it is the most valuable part of the paper.

In football 2v2, **neither approach reproduced the demonstration.** DQAAS learned to pass and shoot *without moving toward the goal*; DQN learned to move toward goal *without passing or shooting*. The demonstration did both.

On 4v8 they test whether the cause is algorithmic — decentralised vs centralised, classic vs recent deep RL — and conclude it is not:

> the cause of the reproducibility issue may not be the centralized/decentralized or classic/recent deep RL

They attribute it instead to **"the domain-specific modeling and reality of the simulator"**, and leave it to future work.

**That is a held source diagnosing the football-RL bottleneck as simulator fidelity rather than algorithm choice** — which directly supports the correction made to [[reinforcement-learning]] and [[imitation-learning]] on 2026-08-07, when the vault weakened its "no simulator" claim to "no evidence of transfer". See [[domain-adaptation]].

## The Prediction That Failed

> **Recorded prominently because the vault made a specific forecast about an unheld source and it was wrong.**

Three pages state that this paper "would presumably settle" whether $\lambda_1$ — the action-supervision weight — is load-bearing. What the paper actually does:

- **Reports no value for $\lambda_1$ or $\lambda_2$ at all.** The text says only that "the $\lambda$ parameters control the weight of these losses."
- **Runs an ablation over *which losses are present*** (DQN / DQfD / DQfAD / DQAS / DQAAS), not over their weights.
- Never varies $\lambda_1$ continuously, and reports no curve.

So the dedicated methodological paper on action supervision is **less specific about the parameter than the applied paper that cites it.** Nakahara et al. at least state $\lambda_1 = 0.01$, $\lambda_2 = 0.1$ and compare against 0.

The $\lambda_1$ question is **unchanged and now harder to answer**, because the obvious source has been checked and does not contain the answer. See [[free-parameters-load-bearing]].

## What It Does Supply Instead — and It Is Arguably Better

The paper measures the **trade-off** that `optimality-gap-is-tunable` was asserted without evidence for.

> The obtained rewards and DTW distance had a trade-off relationship. In general, how to strike a balance is not obvious.

And, on chase-and-escape:

> with increased training steps, the DQAAS first learned the ability to maximize a reward and then learned the reproducibility at the expense of the reward

**Imitation fidelity and reward maximisation genuinely trade off, and the position on that frontier moves with training time.** Not with $\lambda_1$ — but the vault's claim was that a *regularisation-toward-observed-behaviour* dial exists, and here it is, in a different parameterisation. See [[imitation-reward-tradeoff]].

The claim survives in modified form: **the dial is real and measured; which knob turns it is not $\lambda_1$ alone.**

## NFootball

The authors **built their own simulator** rather than use [[google-research-football|GFootball]], because GFootball's "transition algorithms are difficult to customize and some commands (e.g., pass) did not work well within our intended timings."

Pitch dimensions copied from GFootball; everything in Python for transparency. See [[nfootball]].

## Reward Design

Goal +10, concede −10, ball gain +1, ball lost −1, out of pitch −5. Plus **a shot reward of +1 added explicitly "because the goal reward was sparse and limited"** — the [[rare-event-proxy-targets|Fujii-group signature]] appearing again, this time inside a reward function rather than a classifier target.

Note that attackers *and* defenders are both rewarded and punished, so unlike almost everything else in the vault this is not an attacking-only framework.

## Limitations

- **Tiny target-RL samples.** 16 training and 5 test episodes in football; 10 test episodes in chase-and-escape.
- **2v2 and 4v8, not 11v11**, acknowledged as computationally out of reach.
- **Reproducibility not achieved** on the football task, acknowledged.
- **A bespoke simulator** means no comparison against GFootball-based work is possible. See [[nfootball]].
- **$\lambda$ values unreported.**
- **Zero variance across seeds** on football, unexplained.
- **Defenders fixed** during attacker training, so the interaction the multi-agent framing promises is only partly exercised.

## What This Adds to the Vault

1. **The Real-to-Sim framing**, and a name for the gap the vault had been describing without one.
2. **[[dynamic-time-warping|DTW]]** as both loss mechanism and evaluation metric.
3. **The imitation/reward trade-off, measured** — the empirical backing `optimality-gap-is-tunable` lacked.
4. **A held source attributing football-RL failure to simulator fidelity**, confirming a vault correction made on inference alone.
5. **The DQfD lineage and its failure at this data scale**, which explains why Nakahara et al. use the loss they use.
6. **A correction to the vault's own forecasting** — see the Prediction That Failed.

## See Also

- [[action-supervision]] · [[domain-adaptation]] · [[dynamic-time-warping]] · [[deep-q-network]] · [[imitation-reward-tradeoff]]
- [[multi-agent-reinforcement-learning]] · [[reinforcement-learning]] · [[temporal-difference-learning]] · [[imitation-learning]] · [[action-space-design]]
- [[nfootball]] · [[google-research-football]] · [[rare-event-proxy-targets]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[atom-scott]] · [[hiroshi-nakahara]] · [[naoya-takeishi]] · [[yoshinobu-kawahara]]
- [[nagoya-university]] · [[university-of-tokyo]] · [[osaka-university]] · [[data-stadium]]
- [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
