---
title: "C-OBSO (Creating Off-Ball Scoring Opportunity)"
type: concept
tags: [off-ball, space-creation, sports-analytics, player-evaluation, counterfactual, trajectory-prediction, optical-tracking-data, action-valuation, theory-based-modelling, construct-validity]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 72%
  inferred: 20%
  generated: 6%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# C-OBSO (Creating Off-Ball Scoring Opportunity)

[[creating-scoring-opportunities-trajectory-prediction|Teranishi, Tsutsui, Takeda & Fujii's]] metric for the player whose **movement improves someone else's** scoring chance. The vault's only framework that assigns value *relationally* — from the mover to the beneficiary — and its only quantitative treatment of [[space-creation]].

## The Construction

For off-ball player $i$ and the eventual ball carrier $k$:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

- $V^k_{OBSO}$ — the shooter's [[obso|OBSO]] in the situation that actually occurred
- $V'^k_{OBSO}$ — the shooter's OBSO in a counterfactual where player $i$ moved as a **[[trajectory-prediction|predicted reference trajectory]]** says an average player would have

So $i$ is credited with *the improvement in $k$'s scoring chance attributable to $i$ deviating from expected movement.*

Three ingredients: a modified [[obso|OBSO]] with a defender-aware score model, a GVRNN trajectory predictor trained on opponent data via [[imitation-learning]], and the difference between them.

## The Modified Score Model

[[obso|OBSO's]] original score term is distance-to-goal alone — the weakest of its three components, and one Spearman himself flags for replacement.

C-OBSO substitutes a **[[theory-based-modelling|theory-based]] shot-blocking model**: shot value computed per-degree across the shooting angle, discounted by a Gaussian PDF on each goal-side defender, with variance widening in distance and **goalkeepers weighted double**. RMSE 0.309 against 0.324 ($p < 10^{-10}$).

The qualitative gain is larger than the RMSE suggests: two shots from equal distance score identically under the original and 0.0489 against 0.1202 under the replacement, separated by defender congestion.

### How Yeung & Fujii Revised It Again

[[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] build on this block model and state four changes, which together read as a useful critique:

| Change | Reason |
|---|---|
| **Exclude the goalkeeper** | Their target is *shot on target*, and a save still counts — so the keeper is irrelevant |
| **Continuous rather than discrete angle** | More precise PDF evaluation |
| **Truncated normal, not normal** | A defender's reach is bounded, not infinite |
| **Sequential event space** | If one defender blocks, later ones cannot; C-OBSO treats them independently |

The first is the instructive one. C-OBSO weights the goalkeeper *double*; Yeung & Fujii remove him entirely. Neither is wrong — the target changed, so which players matter changed with it. See [[rare-event-proxy-targets]].

The third and fourth are corrections that apply to C-OBSO on its own terms: an untruncated normal gives a defender unbounded reach, and independent defenders can jointly exceed unit probability.

## The Odd Property at Its Centre

**If prediction were perfect, C-OBSO would be identically zero.** The authors state this outright.

The metric measures deviation from a *particular model's* expectation, so it depends on that model being imperfect. Improving the trajectory predictor shrinks the metric. It also means values are not portable — a C-OBSO computed with a different predictor is a different quantity.

## Results

Against annual salary, 15 Yokohama players with ≥10 evaluated sequences:

| Metric | ρ with salary | p |
|---|---|---|
| **C-OBSO** | **0.45** | **0.046** |
| [[obso\|OBSO]] | −0.28 | 0.154 |
| Goals | −0.23 | 0.208 |

**On the same players, neither a player's own off-ball scoring opportunity nor his goal tally relates to salary at all** — only the space he creates for others does. Being the only positive result among three tested on one sample makes it harder to dismiss as fishing.

### The expert-ratings table, read carefully

| Player | ρ (C-OBSO vs rating) | ρ (goals vs rating) |
|---|---|---|
| Nakagawa (season MVP) | **0.75** | 0.63 |
| Marcos | 0.27 (ns) | 0.71 |
| Edigar | −0.37 (ns) | 0.91 |

**Goals predict expert ratings strongly for everyone; C-OBSO only for the MVP.** Seven further players show no correlation. The generous reading is that ratings are largely goal-driven and C-OBSO measures what raters ignore; the sceptical reading is that one significant result in eight is chance. The evidence does not settle it.

## C-OBSO Has Now Been Compared Against Another Off-Ball Metric

> **Added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] — **the first external check this metric has received.**

Nakahara et al.'s [[multi-agent-reinforcement-learning|multi-agent RL]] Q-values are computed on **the same club, the same season, the same [[data-stadium|provider]] and essentially the same 14 players**. The correlation:

$$\rho = 0.182,\quad p > 0.05$$

**No relationship.** Two metrics, from the same research group, both presented as measures of off-ball contribution, on the same data, are statistically unrelated.

### What the source says, and what it does not

Nakahara et al. read this benignly: C-OBSO ranks forwards highly (Edigar, Nakagawa), their Q-values rank midfielders and defenders highly (Kida with 0 season goals, Hirose with 1), so the two capture different aspects of off-ball play.

That reading is available and probably partly right. But it **concedes something the paper does not follow through on**: if C-OBSO ranks forwards and Q-values rank defenders, then C-OBSO is not measuring "off-ball contribution" — it is measuring *space creation for a shooter*, which is a narrower and more forward-weighted construct than its name implies.

That narrower reading is in fact **exactly what the construction says it measures.** C-OBSO is defined only on shot-ending sequences, credits improvement in a *shooter's* OBSO, and clips negative values. The disagreement with Q-values is not a surprise on the mathematics; it is a surprise only on the naming. See [[construct-validity]].

### Why this matters more than the number

$N = 14$. The correlation is weak evidence of anything. Its significance is that **it exists at all** — it is the only head-to-head between two off-ball metrics in the vault, and the field's stock of such comparisons is now one. See [[off-ball-value]].

It also creates a specific, cheap next step. The same dataset would support a full correlation matrix over C-OBSO, Q-values, [[obso|OBSO]] and [[space-occupation-gain|SOG]], which would say whether off-ball value has one factor or several. Nobody has run it.

## Limitations

- **Negative values clipped to zero**, because the *predicted* defender often behaves implausibly. The metric therefore cannot penalise bad movement.
- **Three of 22 players predicted**, for computational reasons.
- **Values are tiny** (0.001–0.01) on no interpretable scale.
- **One team, 34 games, one season.**
- **Salary is heavily confounded** by age, position, nationality, contract timing.
- **Only shot-ending sequences** (412), so movement creating chances that never become shots is invisible. **This is now the limitation that best explains the $\rho = 0.182$ result** — Nakahara et al. evaluate all attacking-third possessions regardless of outcome.
- **No [[split-half-reliability|reliability]] figure.** Neither does its only comparator, so the disagreement above cannot be attributed between "different constructs" and "one of them is unstable".

## Where It Sits

| | [[obso]] | **C-OBSO** | [[vdep]] | [[xsot\|xOSOT]] | Nakahara $Q$ |
|---|---|---|---|---|---|
| Values | The receiver | **The creator** | The defence | Best passing option | The mover |
| Unit | Player | **Player** | Team | Situation | Player |
| Mechanism | Surface read at position | **Counterfactual difference** | Classifier on 22 positions | Max over teammates | Learned [[temporal-difference-learning\|TD]] action-value |
| Needs a baseline | Yes | **Yes** | No | Yes | **No** |
| Sequences used | All | **Shot-ending only** | All | Shot situations | All attacking-third |

C-OBSO and [[vdep]] were the two Fujii-group answers to off-ball valuation, taking opposite routes — VDEP puts everything in the model state and gets a team number; C-OBSO intervenes on one player and gets an individual number. **The individuating ingredient is the counterfactual, not the data.** See [[counterfactual-baseline]].

**A third route now exists.** Nakahara et al. individuate by **agent decomposition** rather than by intervention — one value function per player, no baseline. That weakens the claim above, which is recorded on [[off-ball-value]].

## See Also

- [[obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[imitation-learning]]
- [[theory-based-modelling]] · [[xsot]] · [[off-ball-value]] · [[vdep]] · [[defensive-valuation]] · [[pitch-control]]
- [[construct-validity]] · [[split-half-reliability]] · [[multi-agent-reinforcement-learning]] · [[action-space-design]] · [[space-occupation-gain]]
- [[masakiyo-teranishi]] · [[keisuke-fujii]] · [[william-spearman]] · [[calvin-yeung]] · [[hiroshi-nakahara]] · [[data-stadium]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
