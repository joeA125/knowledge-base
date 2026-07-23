---
title: "Action Valuation Frameworks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, reliability, interpretability, markov-model, point-process]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Action Valuation Frameworks Compared

The vault holds four frameworks for valuing what players do: [[expected-goals|xG]], [[expected-threat|xT]], [[vaep]], and basketball's [[martingale-epv]]. This page compares them across the design axes that actually distinguish them.

For the shared idea underlying xT and martingale EPV specifically, see [[expected-possession-value]]; for the general task, [[action-valuation]].

## The Shared Skeleton

All four instantiate the same equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

They differ only in what $S$ contains and how $Q$ is computed. Everything below follows from those two choices.

## Side-by-Side

| | [[expected-goals\|xG]] | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Martingale EPV]] |
|---|---|---|---|---|
| **Sport** | Soccer | Soccer | Soccer | Basketball |
| **Data** | [[event-stream-data\|Event stream]] | Event stream | Event stream | [[optical-tracking-data\|Optical tracking]] |
| **State $S$** | Shot features | Ball's zone | Last 3 actions + game context | Full tracking history $\mathcal{F}_t$ |
| **Estimation of $Q$** | Classifier | [[value-iteration]] | [[gradient-boosting]] classifiers | Bayesian [[multiresolution-modelling\|multiresolution]] process |
| **Actions valued** | Shots only | Ball progression | All 21 [[spadl]] types | All on-ball events |
| **Horizon** | Immediate | Current possession | Next $k=10$ actions | End of possession |
| **Models risk (conceding)?** | No | No | Yes | Implicitly (turnover states) |
| **[[martingale]] guarantee** | — | No | No | Yes |
| **[[interpretability]]** | Moderate | **High** | Low | Low |
| **[[split-half-reliability\|Reliability]]** | — | **ρ = 0.89** | ρ = 0.25 | Not reported |
| **Cost** | Trivial | Trivial | Modest | 461 processors |

## The Central Trade-off

There is a consistent pattern running through the table: **richer state representations buy sensitivity and pay in stability, interpretability, and cost.**

- xT's state is a single zone. It cannot see risk, context, or finishing — but its ratings are extraordinarily stable ($\rho = 0.89$) and the whole model is a visualisable heatmap.
- VAEP's state is three actions plus game context. It captures risk, values every action type, and responds to context — but its player ratings barely replicate across halves of a season ($\rho = 0.25$), and no individual valuation can be readily explained.
- Martingale EPV's state is the entire tracking history. It sees off-ball positioning and counterfactual passing options that neither soccer model can — at the price of a model so expensive that, as its authors note, it likely confines the approach to academia and well-resourced professional teams.

Critically, [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] show the reliability gap is not just about scope: restricting VAEP to xT's action set and offensive dimension only recovers $\rho = 0.25 \to 0.59$, still far below 0.89. The richer representation *itself* introduces variance.

## A Different Axis: Valuation vs Forecasting

All four frameworks above value actions that *already happened*. [[nmstpp]] does something orthogonal — it **forecasts** the next event's time, location, and action type, with valuation ([[hpus]]) following only downstream.

This distinction matters because the two tasks have different requirements:

| | Valuation frameworks | Forecasting ([[nmstpp]]) |
|---|---|---|
| Question | How good was that action? | What happens next? |
| Needs outcome labels | Yes (goals, points) | No |
| Models event *timing* | No (xT, VAEP) | **Yes** |
| Unit of analysis | Action → player | Event → possession → team |

[[hpus]] is the striking case: it correlates −0.78 with final league position **without ever using goal or shot-outcome data**, in either the model or the metric. Every valuation framework in the table above depends on outcome labels somewhere. That HPUS achieves comparable signal from event *dynamics* alone suggests the two approaches are picking up substantially overlapping information by different routes.

The cost is that HPUS is possession- and team-level, so it cannot do the player valuation that is xT and VAEP's primary purpose.

[[martingale-epv|Martingale EPV]] sits between the camps: it forecasts continuously (via [[competing-risks]] hazards) *and* values, which is why it needs both a [[point-process]] treatment of event timing and a [[martingale]] structure for valuation.

## What Each Rewards

The models disagree about players in ways that trace directly to design choices:

| Player | Divergence | Cause |
|---|---|---|
| Sergio Agüero | 19th VAEP, 109th xT | Elite finisher; xT gives zero credit for shooting |
| Alexis Sánchez | 7th xT, 106th VAEP | Creates threat without finishing it |
| Virgil van Dijk | 81st VAEP, 142nd xT | Defender; *both* frameworks structurally favour attackers |

Correlations with traditional metrics confirm the tendencies: VAEP tracks goals/90 more closely ($\rho = 0.41$ vs 0.26), xT tracks assists/90 more closely ($\rho = 0.53$ vs 0.33).

## A Terminology Warning

"Expected possession value" means two different things in this literature:

- In **basketball**, it names [[martingale-epv|Cervone et al.'s]] specific continuous-time martingale construction from tracking data.
- In **soccer**, "EPV approaches" is a *category label* for possession-based Markov models — the family containing xT, Rudd (2011), Mackay (2017), and Yam (2019).

See [[expected-possession-value]] for the umbrella concept and the distinction.

## Structural Limitations Shared by All

1. **Offensive bias.** Value is defined by proximity to scoring, so defensive contribution is systematically undervalued in every framework — including NMSTPP, which models only the possessing team's on-ball events.
2. **On-ball only** (except tracking-based models, partially). Pressing, marking, and off-ball movement are invisible to all the soccer models.
3. **No ground truth.** When xT and VAEP disagree about a through ball, there is no way to adjudicate. The Van Roy paper is explicit that determining true action values is "very difficult, if not impossible."
4. **Context-dependence of ratings.** It is easier to accumulate value in a weaker league or a stronger team, so cross-context comparison remains unresolved.

## Practical Guidance

- **Season-long recruitment and scouting** → xT's reliability is a strong argument; unstable ratings are actively misleading when the decision is expensive.
- **Analysing specific passages of play** → VAEP's context sensitivity is the point, and season-level reliability is not the relevant criterion.
- **Team-level possession quality, or where outcome data is sparse/unavailable** → [[hpus]], which needs no goal data at all.
- **Tactical and off-ball analysis with tracking data available** → martingale-EPV-style models, accepting the cost.
- **Shot quality alone** → xG remains the right tool, and is a component of both xT and VAEP rather than a competitor.

## See Also

- [[action-valuation]]
- [[expected-possession-value]]
- [[expected-threat]]
- [[vaep]]
- [[martingale-epv]]
- [[expected-goals]]
- [[hpus]]
- [[nmstpp]]
- [[split-half-reliability]]
- [[markov-game]]
