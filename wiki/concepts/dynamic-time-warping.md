---
title: "Dynamic Time Warping"
type: concept
tags: [sequence-alignment, time-series, trajectory-prediction, evaluation, machine-learning, dynamic-programming, domain-adaptation, sports-analytics, speech-recognition, auxiliary-loss]
sources: [raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.8
provenance:
  extracted: 48%
  inferred: 26%
  generated: 6%
  imported: 20%
  ambiguous: 0%
lifecycle: draft
created: 2026-08-07
updated: 2026-08-07
---

# Dynamic Time Warping

An algorithm for matching two sequences that may run at different rates, by finding a **warping path** — a monotonic correspondence between their indices — that minimises total distance between matched elements. Vintsyuk (1968); popularised for speech by Sakoe & Chiba (1978).

Given sequences $s = s_1 \dots s_n$ and $s^E = s_1^E \dots s_m^E$, DTW builds a local distance matrix $d(s,s^E) \in \mathbb{R}^{n \times m}$ (typically Euclidean) and accumulates a warping-path matrix $W(s,s^E)$ under constraints of **monotonicity, continuity, and boundary** — the path may not go backwards, may not skip, and must join corner to corner.

It is [[value-iteration|dynamic programming]]: the optimal path to $(i,j)$ is the best of the three predecessors plus the local cost, which is the same recursive-decomposition move used by [[expected-threat|xT]] on a zone grid, applied to alignment rather than value.

## Why It Is Here

[[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] use DTW **twice over, for two different jobs**, and keeping them apart is the point of this page.

### 1. As an alignment mechanism inside a loss

Plain [[action-supervision]] supervises the agent's Q-values at timestep $t$ against the expert's action at timestep $t$. That assumes agent and demonstration are **in phase** — which holds when working purely inverse on real data, and fails as soon as the agent rolls out in a simulator with different dynamics.

The fix:

$$t' = \underset{j}{\arg\min}\; W(s, s^E)_{t,j}$$

Supervise against $a_{t'}^E$: the expert action at the **most similar state**, not the contemporaneous one. See [[domain-adaptation]].

> ### `alignment-should-precede-supervision`
> **Wherever a model is supervised against a reference trajectory that may run at a different rate, index correspondence is an unstated assumption of matched dynamics. Aligning first is the correction, and it is rarely done.**
> ^[generated: no source states this general form; drawn here from the paper's motivation for $J_{AS+DA}$. rests-on: source:fujii-dtw-supervision-motivation]

This has teeth beyond RL. [[c-obso|C-OBSO]] differences a player's realised trajectory against a **predicted** one frame by frame, and [[trajectory-prediction|GVRNN]] rollouts and real movement need not stay in phase either. Whether C-OBSO's values would change under DTW alignment is untested and cheap to check.

### 2. As an evaluation metric

DTW distance between simulated and demonstrated trajectories is the paper's measure of **reproducibility as imitation** — one half of its two-metric evaluation, the other being reward obtained.

The authors justify the choice on legibility: it is well known and makes it easy to verify whether learning succeeded. That is a modest and honest reason, and it matters because the vault's off-ball metrics mostly lack any such observable target. See [[construct-validity]].

**Using one instrument for both the training signal and the evaluation is worth flagging.** The supervision loss aligns via DTW and the evaluation scores via DTW distance, so a model that games the alignment gets credit twice. The paper does not discuss this. It is mitigated by the two metrics pulling against each other — see [[imitation-reward-tradeoff]] — but not eliminated.

## What DTW Can and Cannot Fix

The paper's own results draw the boundary cleanly, and it is the most useful thing here.

| Task | Domain gap | Does DTW help? |
|---|---|---|
| Chase-and-escape | Predator mobility 120% → 110% | **Yes** — DQAAS beats DQAS on DTW distance |
| Football 2v2, 4v8 | Real players → simulator | **No** — DQAS and DQAAS identical, zero standard error |

DTW corrects **phase** mismatch: the same behaviour occurring at a different rate. A mobility ratio produces exactly that. The football gap is not primarily phase — it is that the demonstrated behaviour (running toward goal *while* passing) is not achievable in the simulator at all.

**Alignment cannot repair a target environment in which the reference behaviour is unavailable.** That is a general limit on the technique and the paper's clearest negative result.

## Properties Worth Knowing

- **Not a metric.** DTW distance violates the triangle inequality, so it cannot be used directly where metric structure is assumed (clustering with centroid methods, kernel constructions without care).
- **Quadratic cost**, $O(nm)$, before banding constraints. The Sakoe–Chiba band is the standard mitigation.
- **Sensitive to the local distance choice.** Euclidean on raw state is what the paper uses; for multi-agent state that means all agents' coordinates weighted equally, which embeds an assumption that every agent's position matters equally to similarity.
- **Degenerate alignments** are possible — one element matched to a long run of another — unless slope constraints are imposed. The paper cites the standard constraint set but does not report which band, if any, was used.

## Beyond Sport

Speech recognition is the origin and still the canonical use. It generalises to any comparison of sequences whose *shape* matters more than their *timing*: gesture recognition, handwriting, gait analysis, ECG morphology, comparing an executed manufacturing process against a reference one.

The recurring pattern is the one above: **DTW turns "did the same thing happen?" into a tractable question when "did it happen at the same time?" is the wrong question.**

## See Also

- [[action-supervision]] · [[domain-adaptation]] · [[imitation-reward-tradeoff]] · [[deep-q-network]] · [[imitation-learning]]
- [[trajectory-prediction]] · [[c-obso]] · [[smoothing]] · [[player-rating-time-series]] · [[performance-volatility]]
- [[value-iteration]] · [[expected-threat]] · [[construct-validity]] · [[jaccard-index]] · [[adjusted-rand-index]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[nagoya-university]] · [[nfootball]]
- [[adaptive-action-supervision-multi-agent-rl|Source Summary]]
