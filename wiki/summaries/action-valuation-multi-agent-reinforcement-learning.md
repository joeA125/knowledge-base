---
title: "Action Valuation of On- and Off-Ball Soccer Players Based on Multi-Agent Deep Reinforcement Learning"
type: source_summary
tags: [summary, reinforcement-learning, multi-agent, action-space, auxiliary-loss, temporal-difference, action-valuation, off-ball, player-evaluation, optical-tracking-data, counterfactual, rnn, sports-analytics, construct-validity, discounting]
sources: [raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.85
provenance:
  extracted: 74%
  inferred: 20%
  generated: 5%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-08-07
updated: 2026-08-07
---

# Action Valuation of On- and Off-Ball Soccer Players Based on Multi-Agent Deep RL

**Nakahara, Tsutsui, Takeda & Fujii** — [[hiroshi-nakahara]], [[kazushi-tsutsui]], [[kazuya-takeda]], [[keisuke-fujii]]. [[nagoya-university]], with RIKEN AIP and JST PRESTO affiliations. arXiv:2305.17886.

**This paper was previously recorded in the vault as "cited, not held"** on [[reinforcement-learning]] and [[keisuke-fujii]]. Both entries are now corrected. It is the **seventh** held source with Fujii as senior author.

## The Claim

Prior [[reinforcement-learning|RL]] valuation in football treats **the team as a single agent** and values only the ball-holder, only at discrete events. This paper values **every attacking player at every timestep**, on and off the ball, by learning a per-player $Q(s,a)$ over a discrete action set — and because $Q$ is defined over the whole action set, it values **actions nobody took**.

## Method

| Component | Choice |
|---|---|
| Agents | **10 independent agents** — one per outfield attacker, goalkeeper excluded. See [[multi-agent-reinforcement-learning]] |
| State $s$ | 22 players + ball, position and velocity — $23 \times 2 \times 2 = 92$ dims |
| Action $a$ | **14 discrete actions** — 8 movement directions at 45°, idle, sprint start, sprint stop, direction release, pass, shot. See [[action-space-design]] |
| Algorithm | **SARSA** — on-policy [[temporal-difference-learning\|TD]], model-free |
| Network | [[gated-recurrent-unit\|GRU]], single 64-unit hidden layer, ReLU, 14 Q-values out |
| Optimiser | [[adam-optimizer\|Adam]], defaults |

### The reward, and what it borrows

Reward is applied **only at the final timestep $T$** of a possession, and is a sum of three terms:

1. **Goal** — 1 if the possession ends in a score
2. **Expected score with no goal** — the [[expected-possession-value|EPV]] of the ball's $(x,y)$ at $T$, cited to [[expected-value-possession-framework|Fernández, Bornn & Cervone]]
3. **Conceding** — $-1$ if the opponent scores immediately after the possession ends

Term 2 is the interesting one: **an existing valuation model is used as the reward function of a new one.** That makes this framework dependent on EPV's correctness in a way none of the group's other work is, and it means the Q-values inherit whatever EPV gets wrong. Term 3 is a rare instance in this vault of a framework whose target is not purely attacking — see [[defensive-valuation]].

### The discount factor

$\gamma = 1$ — **no discounting at all**, "for simplicity", citing Liu et al. (2020). Notable given how much weight [[temporal-discounting|Shelopugin's $\gamma = 0.95$/s]] carries elsewhere in the vault. Since reward arrives only at $T$ and episodes are capped at 300 frames, undiscounted return is finite, so this is defensible — but it means credit is spread **flat** across a possession of up to 30 seconds.

### Action supervision — the contribution

Plain TD learning on this data underfits: most of the 14 actions are rarely taken in any given state, so their Q-values are barely constrained. The fix is an [[action-supervision|auxiliary cross-entropy loss]] treating the softmax over $Q$ as a policy and the observed action as its label:

$$\mathcal{L}_{AS} = -\sum_{t \in T} \mathbf{a}_t \cdot \log\left(\text{softmax}(\mathbf{Q}_{s_t})\right)$$

$$\mathcal{L}_{total} = \mathcal{L}_{TD} + \lambda_1 \mathcal{L}_{AS} + \lambda_2 \mathcal{L}_{L_1}$$

with $\lambda_1 = 0.01$, $\lambda_2 = 0.1$.

The stated assumption is **that real players act better than random** — an [[imitation-learning|imitation]] prior injected into a value-learning objective. The authors are explicit about the trade-off it creates: too little supervision and counterfactual action values are never learned; too much and the model **overfits to the actions actually taken and stops considering counterfactuals**. See [[observed-versus-optimal-decisions]], where this turns out to matter well beyond this paper.

## Data

54 games from the **Meiji J1 League 2019**, including all 34 [[data-stadium|Data Stadium]] tracked games of Yokohama F. Marinos. Tracking at 25 Hz downsampled to 10 Hz; event data at 30 Hz. Possessions split from ball recovery to ball loss or goal, 50–300 frames, **attacking third only**.

Train: 1,669 non-Yokohama sequences (186 validation). Test: 1,236 Yokohama sequences.

**Same league, same season, same provider and the same club as [[c-obso|C-OBSO]]** — so the two are computed on essentially the same players, which is what makes the comparison below possible at all.

## Results

### The ablation

| | $\mathcal{L}_{TD}$ | $\mathcal{L}_{AS}$ | Mean on-ball $Q$ | Mean off-ball $Q$ |
|---|---|---|---|---|
| With supervision | **0.0034 ± 0.0001** | 3.9550 | **0.0137** | **0.0008** |
| Without | 0.0063 ± 0.0004 | **3.9407** | 0.0008 | $-7.123$ |

Supervision halves TD loss and **slightly worsens the supervised loss itself** — an odd result the authors do not explain. Qualitatively, the supervised model concentrates Q-mass on the action actually chosen (a pass, in their worked example); the unsupervised model spreads near-zero values across all 14.

The authors state plainly that **they cannot determine which model is better**, because there is no ground truth for a Q-value.

### Correlations with existing indicators

$N = 14$ players with ≥10 games. Spearman's $\rho$, treated as an **effect size, not a significance test**, given the sample.

| Compared against | $\rho$ | Reading |
|---|---|---|
| Season goals | **−0.761** | Strong *negative* |
| Expert match ratings | −0.218 | Low negative |
| [[obso\|OBSO]] | −0.305 | Low negative |
| [[c-obso\|C-OBSO]] | 0.182 | **None** |
| Own on-ball $Q$ vs own off-ball $Q$ | 0.618 | Moderate positive |

**The headline correlation is negative and the authors treat that as success.** Their argument: the metric rewards defenders and midfielders who make many passes and move well off the ball. Thiago (defender, 0 goals) and Hatanaka (defender, 0 goals, 2nd-most passes in the league) top the ranking; Marcos and Nakagawa (15 goals each) sit near the bottom.

This is [[construct-validity|construct validation by divergence]] — the claim is not that the metric agrees with goals but that it *shouldn't*, because goals are exactly what off-ball value is meant to be invisible to.

### The C-OBSO comparison is the most valuable result here

$\rho = 0.182$, $p > 0.05$ — **no relationship**. Two off-ball metrics, from the same research group, on the same club, season and data provider, are **statistically unrelated**.

The paper reads this benignly: C-OBSO favours forwards (Edigar, Nakagawa), Q-values favour midfielders and defenders (Kida, Hirose), so they measure different things. That reading is available. So is a harsher one, which the paper does not consider: **two metrics both claiming to measure "off-ball contribution" and correlating at 0.18 cannot both be measuring off-ball contribution** — at least one is measuring something narrower than its name.

This is the first head-to-head comparison of two off-ball metrics anywhere in the vault. See [[off-ball-value]] and [[construct-validity]].

## Limitations

- **$N = 14$ players, one club, one season.** Every correlation rests on 14 points.
- **No ground truth**, acknowledged. Validation is entirely by correlation with other metrics.
- **Attacking third only** — off-ball movement in build-up is excluded, which sits oddly with a metric whose stated virtue is valuing players far from the ball.
- **Independent agents.** Agents do not model each other; the "multi-agent" claim is about *count*, not interaction. See [[multi-agent-reinforcement-learning]].
- **Q-values are tiny** (0.0008–0.02) on no interpretable scale — the same complaint the vault records against [[c-obso|C-OBSO]].
- **On-ball actions are impoverished.** Dribbling and trapping labels exist in the data and were discarded "for simplicity", so the on-ball action set is pass, shot and movement only.
- **Reward depends on EPV**, an external model whose errors propagate silently.
- **No [[split-half-reliability|reliability]] figure** — consistent with every other off-ball metric held here.
- **$\lambda_1$ is unswept.** Two points (0 and 0.01) are compared, and the authors describe the behaviour at both extremes without measuring it. See [[free-parameters-load-bearing]].

## Free Parameters Introduced

| Parameter | Value | Role | Justification |
|---|---|---|---|
| $\lambda_1$ | 0.01 | Action-supervision weight | Asserted; two-point ablation only |
| $\lambda_2$ | 0.1 | $L_1$ weight | Asserted |
| $\gamma$ | 1 | Discount | "Simplicity", citing Liu et al. |
| $T_{max}$ | 300 frames (30 s) | Episode cap | Asserted |
| $T_{min}$ | 50 frames | Episode floor | Asserted |
| Stop threshold | 0.1 m/s | Gates the "idle" label | Asserted |
| Sprint threshold | 24 km/h | Gates "sprint" labels | Asserted |

The last two are **gates in the [[free-parameters-load-bearing|gate/horizon/shape taxonomy]]** — they determine which action label a frame receives, so a wrong threshold assigns the wrong action, not a slightly wrong value.

## What This Adds to the Vault

1. **The first genuine RL framework in the football-valuation corpus.** Everything else borrows RL vocabulary; this optimises a Bellman residual.
2. **A sixth off-ball mechanism** — learned action-values over all players and timesteps, needing neither a counterfactual reference nor a value surface.
3. **The first head-to-head between two off-ball metrics**, and it comes out at $\rho = 0.18$.
4. **A tunable that controls how far "optimal" may diverge from "observed"** — which reframes [[observed-versus-optimal-decisions]].
5. **A negative correlation with goals presented as a success criterion** — see [[construct-validity]].

## See Also

- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[temporal-difference-learning]] · [[action-space-design]] · [[construct-validity]]
- [[reinforcement-learning]] · [[off-ball-value]] · [[action-valuation]] · [[c-obso]] · [[obso]] · [[expected-possession-value]]
- [[counterfactual-simulation]] · [[imitation-learning]] · [[policy-modelling]] · [[temporal-discounting]] · [[gated-recurrent-unit]]
- [[hiroshi-nakahara]] · [[kazushi-tsutsui]] · [[kazuya-takeda]] · [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[google-research-football]]
- [[nagoya-university]] · [[data-stadium]]
- [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[beyond-expected-goals|OBSO Summary]] · [[expected-value-possession-framework|EPV Summary]]
