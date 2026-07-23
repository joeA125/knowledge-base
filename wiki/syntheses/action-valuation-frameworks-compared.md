---
title: "Action Valuation Frameworks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, reliability, predictive-validity, interpretability, markov-model, point-process, path-signature]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md]
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

The vault holds four frameworks for valuing what players do — [[expected-goals|xG]], [[expected-threat|xT]], [[vaep]], and basketball's [[martingale-epv]] — plus two possession-level metrics derived from forecasting models, [[hpus]] and [[lpv]].

For the shared idea underlying xT and martingale EPV specifically, see [[expected-possession-value]]; for the general task, [[action-valuation]].

## The Shared Skeleton

All four valuation frameworks instantiate the same equation:

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

A consistent pattern: **richer state representations buy sensitivity and pay in stability, interpretability, and cost.**

- xT's state is a single zone. It cannot see risk, context, or finishing — but its ratings are extraordinarily stable ($\rho = 0.89$) and the whole model is a visualisable heatmap.
- VAEP's state is three actions plus game context. It captures risk, values every action type, and responds to context — but its player ratings barely replicate across halves of a season ($\rho = 0.25$).
- Martingale EPV's state is the entire tracking history. It sees off-ball positioning and counterfactual passing options neither soccer model can — at a cost that confines it to academia and well-resourced clubs.

Critically, [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] show the reliability gap is not just about scope: restricting VAEP to xT's action set and offensive dimension only recovers $\rho = 0.25 \to 0.59$. The richer representation *itself* introduces variance.

## A Different Axis: Valuation vs Forecasting

All four frameworks above value actions that *already happened*. A second line of work **forecasts** the next event, with valuation following downstream:

| | [[seq2event]] (2022) | [[nmstpp]] (2023) | [[sig-model]] (2025) |
|---|---|---|---|
| History | Fixed window | Fixed window (40) | **Whole possession** |
| Encoder | [[transformer]] | Transformer | [[path-signature]] + feedforward |
| Forecasts time? | No | **Yes** | No |
| Forecasts location? | Yes | Zone (20) | **Exact $(x,y)$** |
| Handcrafted features | **Required** | Used | **Harmful** |
| Derived metric | poss-util | [[hpus]] | [[lpv]] |

The lineage is a genuine argument, each step objecting to something in the last. NMSTPP says Seq2Event ignores *when*; Sig-Model says both use the wrong *unit*, since a fixed window spans possession boundaries and discards possession length.

The [[feature-engineering]] row captures a finding that generalises: Seq2Event degrades without handcrafted geometry, while Sig-Model degrades *with* it. Engineered features are a crutch for a representation that cannot recover the geometry itself.

Forecasting and valuation have different requirements:

| | Valuation frameworks | Forecasting models |
|---|---|---|
| Question | How good was that action? | What happens next? |
| Needs outcome labels | Yes (goals, points) | No |
| Unit of analysis | Action → player | Event → possession → team |

[[martingale-epv]] sits between the camps — it forecasts continuously via [[competing-risks]] hazards *and* values, which is why it needs both a [[point-process]] treatment of timing and a [[martingale]] structure for valuation.

## Possession Metrics: HPUS vs LPV

| | [[hpus]] | [[lpv]] |
|---|---|---|
| Action value from | Constants 0/5/10 | [[expected-goals\|xG]] and [[expected-threat\|xT]] at predicted location |
| Location | Multiplied in (3 areas) | Built into the value |
| Time | Divides by interevent time | Not modelled |
| Within-possession weighting | Exponential decay | None |
| Interpretable units | No | **Yes** |
| vs next-match xG | 0.27 | **0.32** |

LPV's critique is that HPUS's constants are unexplained and, by its authors' own admission, adjustable — which makes the scale arbitrary and cross-study comparison impossible. LPV answers this by using units the field already understands. The trade is real though: LPV loses the temporal information that was HPUS's distinguishing contribution.

## Metrics Beat Outcomes at Predicting Outcomes

The most striking cross-paper result, and it is not about any single metric:

| | poss-util | HPUS | LPV | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

**Both possession-value metrics predict a team's next match better than xG or goals do — including predicting goals themselves.** Goals are the *worst* predictor of future goals in the table.

A scoreline is a small, noisy sample of an underlying process; a possession metric aggregates hundreds of actions and estimates that process more directly. See [[predictive-validity]].

## What Each Rewards

| Player | Divergence | Cause |
|---|---|---|
| Sergio Agüero | 19th VAEP, 109th xT | Elite finisher; xT gives zero credit for shooting |
| Alexis Sánchez | 7th xT, 106th VAEP | Creates threat without finishing it |
| Virgil van Dijk | 81st VAEP, 142nd xT | Defender; *both* frameworks structurally favour attackers |

VAEP tracks goals/90 more closely ($\rho = 0.41$ vs 0.26); xT tracks assists/90 more closely ($\rho = 0.53$ vs 0.33).

## A Terminology Warning

"Expected possession value" means two different things: in **basketball**, [[martingale-epv|Cervone et al.'s]] specific martingale construction; in **soccer**, a *category label* for possession-based Markov models including xT. See [[expected-possession-value]].

## Structural Limitations Shared by All

1. **Offensive bias.** Value is defined by proximity to scoring, so defensive contribution is systematically undervalued everywhere — including the forecasting models, which model only the possessing team.
2. **On-ball only** (except tracking-based models, partially).
3. **No ground truth.** Nothing adjudicates when two frameworks disagree, which is why [[split-half-reliability|reliability]] and [[predictive-validity]] have become the substitute tests.
4. **Context-dependence.** Accumulating value is easier in a weaker league or a stronger team.

## Practical Guidance

- **Season-long recruitment and scouting** → xT's reliability is a strong argument; unstable ratings mislead when decisions are expensive.
- **Analysing specific passages of play** → VAEP's context sensitivity is the point.
- **Team-level possession quality, or where outcome data is sparse** → LPV (interpretable units) or HPUS (if timing matters).
- **Forecasting the next action** → Sig-Model if location matters and compute is limited; NMSTPP if timing matters.
- **Tactical and off-ball analysis with tracking data** → martingale-EPV-style models, accepting the cost.
- **Shot quality alone** → xG, which is a *component* of xT, VAEP, and LPV rather than a competitor.

## See Also

- [[action-valuation]]
- [[expected-possession-value]]
- [[expected-threat]] · [[vaep]] · [[martingale-epv]] · [[expected-goals]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]]
- [[split-half-reliability]] · [[predictive-validity]] · [[feature-engineering]]
