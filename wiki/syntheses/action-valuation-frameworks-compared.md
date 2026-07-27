---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, counterfactual, clustering, event-prediction, reliability, predictive-validity, time-series, volatility, player-development, recruitment]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md]
confidence: 0.85
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **four distinct tasks**, each answering a different question, with different data requirements and different validation strategies.

*(Formerly "Action Valuation Frameworks Compared" — renamed as the vault outgrew the valuation-only framing.)*

## The Four Tasks

| Task | Question | Unit | Needs outcome labels? | Examples |
|---|---|---|---|---|
| **Valuation** | How good was that action? | Action → player | Yes | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[martingale-epv]] |
| **Forecasting** | What happens next? | Event | No | [[seq2event]], [[nmstpp]], [[sig-model]] |
| **Clustering** | What kind of sequence is this? | Possession | No | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual simulation** | What if this player joined? | Episode, given a lineup | No (uses value as target) | [[scoutgpt]] |

Forecasting models spawn valuation metrics downstream ([[hpus]] from NMSTPP, [[lpv]] from Sig-Model), which is why the two are often conflated — but the modelling target genuinely differs.

**A cross-cutting dimension.** [[football-performance-time-series|Mendes-Neves et al.]] do not add a fifth task; they attack the **aggregation step** shared by all four. Every task above ends by collapsing something over time — actions into a player rating, events into a metric, possessions into cluster proportions. That collapse is treated as a formality everywhere except in their work. See [Time as a Cross-Cutting Axis](#time-as-a-cross-cutting-axis) below.

## Task 1: Valuation

All four valuation frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

differing only in what $S$ contains and how $Q$ is computed.

| | [[expected-goals\|xG]] | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Martingale EPV]] |
|---|---|---|---|---|
| **Data** | [[event-stream-data\|Event]] | Event | Event | [[optical-tracking-data\|Tracking]] |
| **State $S$** | Shot features | Ball's zone | Last 3 actions + context | Full tracking history |
| **Estimation** | Classifier | [[value-iteration]] | [[gradient-boosting]] | Bayesian [[multiresolution-modelling\|multiresolution]] |
| **Models risk** | No | No | Yes | Implicitly |
| **[[martingale]] guarantee** | — | No | No | **Yes** |
| **[[interpretability]]** | Moderate | **High** | Low | Low |
| **[[split-half-reliability\|Reliability]]** | — | **ρ = 0.89** | ρ = 0.25 | Not reported |
| **Cost** | Trivial | Trivial | Modest | 461 processors |

**The central trade-off:** richer state representations buy sensitivity and pay in stability, interpretability, and cost. [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] show the reliability gap is not merely about scope — restricting VAEP to xT's action set only recovers $\rho = 0.25 \to 0.59$, so the richer representation *itself* introduces variance.

Player-level disagreements trace directly to design: Agüero ranks 19th by VAEP and 109th by xT (elite finisher; xT gives no credit for shooting); Sánchez 7th by xT and 106th by VAEP (creates threat without finishing). Van Dijk ranks 81st and 142nd — **both frameworks structurally favour attackers.**

### A Missing Axis: Intent vs Outcome

The table above compares frameworks on state richness, but there is an orthogonal axis they differ on **without ever declaring it**: whether the model can see how the action turned out.

| Framework | Position |
|---|---|
| [[expected-threat\|xT]] | Effectively pure intent — values only zone transition |
| [[expected-goals\|xG]] | Pure intent by construction — chance quality, not strike quality |
| [[vaep]] | Conflated — outcome features present but not isolated |
| **I-VAEP / O-VAEP** | **Explicitly separated** |

Seen this way, several apparent disagreements dissolve. Agüero ranking 19th by VAEP and 109th by xT is not really a dispute about player quality — it is xT measuring intent and VAEP measuring intent-plus-execution, on a player whose value is concentrated in execution. [[intent-vs-outcome-valuation]] makes the axis explicit and cheap to implement.

## Task 2: Forecasting

| | [[seq2event]] (2022) | [[nmstpp]] (2023) | [[sig-model]] (2025) | [[scoutgpt]] (2026) |
|---|---|---|---|---|
| History | Fixed window | Fixed window (40) | **Whole possession** | Episode + lineup |
| Encoder | [[transformer]] | Transformer | [[path-signature]] + FFN | GPT-2 decoder |
| Forecasts time? | No | **Yes** | No | Yes |
| Location | Zone | Zone (20) | **Exact $(x,y)$** | Binned $(x,y)$ |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP |

Each step objects to something in the last: NMSTPP says Seq2Event ignores *when*; Sig-Model says both use the wrong *unit*; ScoutGPT says none of them can be *conditioned on a hypothetical lineup*.

The [[feature-engineering]] row records a finding that generalises: Seq2Event degrades without handcrafted geometry, Sig-Model degrades *with* it. Engineered features are a crutch for a representation that cannot recover the geometry itself.

## Task 3: Clustering

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] fit a [[mixture-model]] whose components are [[point-process|marked spatio-temporal point processes]], clustering whole possessions into tactical types — short direct counter-attacking through to elaborate positional play.

This needs no outcome labels and no notion of value. Validation is by [[adjusted-rand-index|ARI]] on simulated data with known ground truth, plus interpretability of the recovered clusters against known tactical vocabulary.

## Task 4: Counterfactual Simulation

[[scoutgpt]] conditions generation on an explicit lineup, so replacing one player and re-generating estimates how that player would perform in a new tactical context. This is the only task here that addresses **distribution shift** — every other approach extrapolates from observed behaviour in the observed context.

The requirements are strict: generative, long enough horizon to compute value over a whole episode, and *surgical* entity conditioning. ScoutGPT achieves the last by never generating player identity, resolving it deterministically from the lineup.

## Time as a Cross-Cutting Axis

Every task above produces a number that is implicitly **an average over a period**. [[player-rating-time-series|Treating that period as a series instead]] yields quantities none of the four tasks produce:

| Quantity | What it captures | Why an average cannot |
|---|---|---|
| Short-term vs long-term rating | Form against underlying quality | Averaging conflates them by construction |
| [[performance-volatility\|Volatility]] | Week-to-week reliability | Two players with one mean, different risk |
| Per-action-type trajectory | Style *change* | Messi's dribble and pass value moved oppositely over a decade |
| [[player-development-curve\|Development curve]] | Career stage and peak | A 23-year-old and a 30-year-old at equal output are different assets |

**This creates a genuine unresolved conflict with the reliability critique.** [[split-half-reliability]] treats within-season variation as noise and marks VAEP down for it. [[performance-volatility|Volatility analysis]] treats the same variation as signal about the player. Both cannot be wholly right.

The decisive experiment is unrun: does short-term deviation from a player's long-term level **predict** next-match contribution beyond the long-term average? The [[predictive-validity]] table below tests metrics at the team-match level and so does not settle it.

## Metrics Beat Outcomes at Predicting Outcomes

The most striking cross-paper result, spanning tasks:

| | poss-util | [[hpus]] | [[lpv]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

Both possession-value metrics predict a team's next match better than xG or goals do — and **goals are the worst predictor of future goals**. A scoreline is a small, noisy sample of an underlying process; a possession metric aggregates hundreds of actions and estimates that process directly. See [[predictive-validity]].

## How Each Task Is Validated

With no ground truth for "correct" action values, validation strategies differ by task — and this is where the tasks diverge most sharply:

| Task | Validation |
|---|---|
| Valuation | Concurrent correlation with xG/goals; [[split-half-reliability]]; [[predictive-validity]] |
| Forecasting | Held-out likelihood, Brier, [[kl-divergence]] against empirical zone-conditioned distributions |
| Clustering | [[adjusted-rand-index]] on simulated data; BIC; interpretability against tactical vocabulary |
| Counterfactual | Self-to-self reconstruction; out-of-sample transfer prediction against actual next-season performance |
| *Time-series derivatives* | *Weakest — face validity and agreement with prior peak-age estimates only* |

The last row is a real gap. [[performance-volatility|Volatility]] and the [[player-development-curve|PDC]] are validated by looking reasonable and by agreeing with one independent study on peak age. Neither is tested against anything they should predict.

## A Terminology Warning

"Expected possession value" means two things: in **basketball**, [[martingale-epv|Cervone et al.'s]] specific martingale construction; in **soccer**, a *category label* for possession-based Markov models including xT. See [[expected-possession-value]].

## Limitations Shared Across All Four

1. **Offensive bias.** Value is defined by proximity to scoring, so defenders are systematically undervalued — and the forecasting and simulation models mostly represent only the possessing team. Partly a *data* problem: [[event-stream-data|event data]] lacks the context to judge tackles and interceptions at all.
2. **On-ball only**, except partially for tracking-based models. Pressing, marking, and off-ball movement are invisible.
3. **No ground truth**, which is why reliability and predictive validity have become the substitute tests.
4. **Context dependence.** Accumulating value is easier in a weaker league or stronger team — the problem counterfactual simulation is the first to attack directly.
5. **[[selection-bias]] throughout.** Every minutes threshold and games-played filter selects on a performance-correlated variable, and the excluded players — young, fringe, newly transferred — are exactly those recruitment most needs to assess.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[scoutgpt|counterfactual simulation]] if assessing fit at a specific club; [[player-development-curve|PDC]] position to distinguish appreciating from depreciating assets.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP against O-VAEP]].
- **Assessing squad risk rather than mean output** → [[performance-volatility|volatility metrics]], residualised against rating.
- **Analysing passages of play** → VAEP's context sensitivity.
- **Team-level possession quality** → [[lpv]] (interpretable units) or [[hpus]] (if timing matters).
- **Opponent scouting / training design** → [[football-event-sequences-point-process-mixture|possession clustering]].
- **Forecasting the next action** → Sig-Model if location matters, NMSTPP if timing does.
- **Tactical and off-ball analysis with tracking data** → martingale-EPV-style models.
- **Shot quality alone** → xG, a *component* of xT, VAEP and LPV rather than a competitor.

## See Also

- [[action-valuation]] · [[expected-possession-value]] · [[counterfactual-simulation]]
- [[expected-threat]] · [[vaep]] · [[martingale-epv]] · [[expected-goals]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]]
- [[large-event-model]] · [[mixture-model]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]] · [[intent-vs-outcome-valuation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[feature-engineering]] · [[selection-bias]]
