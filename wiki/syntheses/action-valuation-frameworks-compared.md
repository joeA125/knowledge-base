---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, counterfactual, clustering, event-prediction, reliability, predictive-validity, time-series, volatility, player-development, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md]
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
| **Valuation** | How good was that action? | Action → player | Yes | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[martingale-epv]], [[pass-carry-reward\|PCR]] |
| **Forecasting** | What happens next? | Event | No | [[seq2event]], [[nmstpp]], [[sig-model]] |
| **Clustering** | What kind of sequence is this? | Possession | No | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | No (uses value as target) | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |

Forecasting models spawn valuation metrics downstream ([[hpus]] from NMSTPP, [[lpv]] from Sig-Model), which is why the two are often conflated — but the modelling target genuinely differs.

**A cross-cutting dimension.** [[football-performance-time-series|Mendes-Neves et al.]] do not add a fifth task; they attack the **aggregation step** shared by all four. Every task above ends by collapsing something over time — actions into a player rating, events into a metric, possessions into cluster proportions. That collapse is treated as a formality everywhere except in their work. See [Time as a Cross-Cutting Axis](#time-as-a-cross-cutting-axis) below.

## Task 1: Valuation

All valuation frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

differing only in what $S$ contains and how $Q$ is computed.

| | [[expected-goals\|xG]] | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Martingale EPV]] | [[pass-carry-reward\|Shelopugin / PCR]] |
|---|---|---|---|---|---|
| **Data** | [[event-stream-data\|Event]] | Event | Event | [[optical-tracking-data\|Tracking]] | Event |
| **State $S$** | Shot features | Ball's zone | Last 3 actions + context | Full tracking history | Action + preceding action + duel skill |
| **Estimation** | Classifier | [[value-iteration]] | [[gradient-boosting]] | Bayesian [[multiresolution-modelling\|multiresolution]] | [[gradient-boosting\|Gradient boosting]] ×9 |
| **Models risk** | No | No | Yes (10 actions) | Implicitly | **Yes (unbounded)** |
| **Credit decay** | — | — | Hard window | — | **Geometric in time** |
| **Values duels** | No | No | No | No | **Yes** |
| **[[martingale]] guarantee** | — | No | No | **Yes** | No |
| **[[interpretability]]** | Moderate | **High** | Low | Low | Low |
| **[[split-half-reliability\|Reliability]]** | — | **ρ = 0.89** | ρ = 0.25 | Not reported | **Not reported** |
| **Cost** | Trivial | Trivial | Modest | 461 processors | Modest |

**The central trade-off:** richer state representations buy sensitivity and pay in stability, interpretability, and cost. [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] show the reliability gap is not merely about scope — restricting VAEP to xT's action set only recovers $\rho = 0.25 \to 0.59$, so the richer representation *itself* introduces variance.

PCR sits firmly on the rich end of that trade-off — more state, unbounded risk horizon, more models — and its reliability is unreported. Given the pattern above, the prior should be that it inherits VAEP-like instability rather than xT-like stability, and that matters because its intended application is [[recruitment]], where reliability dominates.

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

### A Second Missing Axis: Attributable Possession

Every framework except Shelopugin's assumes it is known **whose** prospects $Q$ describes. That assumption silently excludes aerial and ground duels, where two opposing players contest a ball neither holds — and duels are precisely where physical mismatch decides outcomes.

[[symmetrical-duel-valuation]] closes the gap by valuing a duel via the control action that follows it, and by conditioning the *passer's* reward on the receiver's [[duel-skill-rating|duel ability]]. The [[epv-control-duel-skills-football|Donnarumma case]] quantifies the cost of ignoring it: a duel-blind model values two tactically identical long passes at 0.00075 and 0.00077, while the duel-aware model separates them roughly two-fold.

There is a striking corollary. Van Dijk ranks 81st by VAEP and 142nd by xT, yet tops **both** of Shelopugin's duel-rating tables. The information distinguishing an elite centre-back is present in ordinary event data; the valuation paradigm simply does not use it. That reframes the offensive-bias problem — it is not solely a definitional consequence of valuing proximity to scoring, but partly a choice about which events get modelled.

### Credit Assignment Over Time

A third axis, and the one with the clearest direction of travel:

| Approach | Boundary | Framework |
|---|---|---|
| Whole possession, undecayed | Turnover | Early EPV / xT |
| Fixed $k$-action window | Action count ($k = 10$) | [[vaep]] |
| Capped time decay | 1 min, floored at 5 actions | [[football-performance-time-series\|Mendes-Neves et al.]] |
| Geometric time decay | None — weight → 0 | [[temporal-discounting\|Shelopugin]] |

The progression is from counting actions to measuring elapsed time, on the reasoning that ten one-touch passes and ten recycling passes span very different intervals and should not attract equal credit. The endpoint is notable: once decay is geometric, **the horizon question disappears** — [[possession-risk|risk]] can be summed over unboundedly many subsequent possessions because distant ones contribute nothing. The same argument that licenses discounting in infinite-horizon [[reinforcement-learning]], arrived at from the credit-assignment side.

The cost is a free parameter. $\gamma = 0.95$ per second is asserted and framed as a stylistic choice (0.9 for direct attacking, 0.99 for *tiki-taka*), with no sensitivity analysis. Since $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$, rankings are almost certainly sensitive to it.

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

## Task 4: Counterfactual and Transfer

This task now holds two structurally different approaches, and they are complementary rather than competing.

| | [[scoutgpt\|Generative simulation]] | [[transfer-performance-prediction\|Regression on context]] |
|---|---|---|
| Object modelled | Event sequence | Season aggregate |
| Destination encoded as | Explicit lineup | [[league-strength-rating\|Club/league strength ratings]] |
| Captures tactical interaction | **Yes** | No |
| Handles role change | Implicitly | **No** — acknowledged failure |
| Data required | Full event streams for destination squad | Season aggregates + match results |
| Scales to a market | No | **Yes** |
| Addresses [[selection-bias\|selection]] | Not addressed | Explicitly, heuristically |
| Cost | Substantial | Modest |

[[scoutgpt]] conditions generation on an explicit lineup, so replacing one player and re-generating estimates how that player would perform in a new tactical context. Its requirements are strict: generative, long enough horizon to compute value over a whole episode, and *surgical* entity conditioning — achieved by never generating player identity, resolving it deterministically from the lineup.

Shelopugin's regression asks the same question without generating anything, predicting next-season [[pass-carry-reward|PCR]] from ~600 features including destination club and league ratings, mean opponent rating, league style, and the player's *share* of team output.

The practical reading is sequential: **regression to narrow a market, simulation to discriminate among survivors.** See [[recruitment]].

One asymmetry is worth flagging. The generative papers do not address the fact that observed transfers were **chosen** by clubs forecasting the same quantity — a [[positive-unlabeled-learning|presence-only]] structure that biases training data in a direction no amount of destination conditioning corrects. Shelopugin addresses it, if only with a heuristic shrinkage he describes as inadequate.

## Time as a Cross-Cutting Axis

Every task above produces a number that is implicitly **an average over a period**. [[player-rating-time-series|Treating that period as a series instead]] yields quantities none of the four tasks produce:

| Quantity | What it captures | Why an average cannot |
|---|---|---|
| Short-term vs long-term rating | Form against underlying quality | Averaging conflates them by construction |
| [[performance-volatility\|Volatility]] | Week-to-week reliability | Two players with one mean, different risk |
| Per-action-type trajectory | Style *change* | Messi's dribble and pass value moved oppositely over a decade |
| [[player-development-curve\|Development curve]] | Career stage and peak | A 23-year-old and a 30-year-old at equal output are different assets |

**This creates a genuine unresolved conflict with the reliability critique.** [[split-half-reliability]] treats within-season variation as noise and marks VAEP down for it. [[performance-volatility|Volatility analysis]] treats the same variation as signal about the player. Both cannot be wholly right.

The decisive experiment is unrun: does short-term deviation from a player's long-term level **predict** next-match contribution beyond the long-term average?

There is also a denominator question that runs underneath all of this. Per-90 rates assume clock minutes measure opportunity, and they do not — [[effective-playing-time|effective playing time]] varies by team, scoreline and league, so per-90 silently favours players at high-tempo sides. Only Shelopugin normalises on it, and only he discounts over it, which matters because dead time is causally inert and should not dilute credit.

## Metrics Beat Outcomes at Predicting Outcomes

The most striking cross-paper result, spanning tasks:

| | poss-util | [[hpus]] | [[lpv]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

Both possession-value metrics predict a team's next match better than xG or goals do — and **goals are the worst predictor of future goals**. A scoreline is a small, noisy sample of an underlying process; a possession metric aggregates hundreds of actions and estimates that process directly. See [[predictive-validity]].

### The Player-Level Gap, Narrowed

This vault has previously flagged that the table above is a **team-match** result and does not license player-level conclusions. Shelopugin supplies the first player-level evidence: predicting next-season PCR against a persistence baseline, RMSE 0.053 → 0.033 overall, and 0.061 → 0.037 for players changing both club and league.

The useful finding is in the stratification. **Persistence degrades monotonically with movement** — last season's number predicts worst for exactly the population recruitment cares about — while the model's advantage holds across all five strata.

But the gap is narrowed, not closed. Hirnschall & Bajons predict an *independent* outcome; Shelopugin predicts **the metric's own future value**. Self-prediction establishes that a metric captures something persistent, not that the persistent thing is skill. A metric measuring tactical role rather than quality would score just as well.

The author concedes this directly: no mathematical demonstration that EPV-based metrics track ability is available, and the two proposed alternatives — expert review, and checking shortlists against actual elite-club transfers — are not executed.

## How Each Task Is Validated

With no ground truth for "correct" action values, validation strategies differ by task — and this is where the tasks diverge most sharply:

| Task | Validation |
|---|---|
| Valuation | Concurrent correlation with xG/goals; [[split-half-reliability]]; [[predictive-validity]] |
| Forecasting | Held-out likelihood, Brier, [[kl-divergence]] against empirical zone-conditioned distributions |
| Clustering | [[adjusted-rand-index]] on simulated data; BIC; interpretability against tactical vocabulary |
| Counterfactual | Self-to-self reconstruction; out-of-sample transfer prediction against actual next-season performance |
| Transfer regression | RMSE/MAE against a persistence baseline, **stratified by whether the player moved** |
| *Time-series derivatives* | *Weakest — face validity and agreement with prior peak-age estimates only* |

The stratification in the fifth row should be standard practice and currently is not. ScoutGPT reports an aggregate MAE against a naive baseline without separating movers from stayers, so it is unknown whether its improvement comes from genuine context modelling or from the stay-put majority of its evaluation set.

## A Terminology Warning

"Expected possession value" now means **three** things: in **basketball**, [[martingale-epv|Cervone et al.'s]] specific martingale construction; in **soccer**, a *category label* for possession-based Markov models including xT; and, in Shelopugin, a supervised event-data model targeting accumulated future xG. See [[expected-possession-value]].

## Limitations Shared Across All Tasks

1. **Offensive bias.** Value is defined by proximity to scoring, so defenders are systematically undervalued — and the forecasting and simulation models mostly represent only the possessing team. Partly a *data* problem, but the van Dijk duel-rating result shows it is also partly a modelling choice about which events are represented at all.
2. **On-ball only**, except partially for tracking-based models. Pressing, marking, and off-ball movement are invisible.
3. **No ground truth**, which is why reliability and predictive validity have become the substitute tests — and why self-prediction keeps getting mistaken for validation.
4. **Context dependence.** Accumulating value is easier in a weaker league or stronger team — the problem Task 4 attacks directly, from two directions.
5. **[[selection-bias]] throughout.** Every minutes threshold and games-played filter selects on a performance-correlated variable, and the excluded players — young, fringe, newly transferred — are exactly those recruitment most needs to assess. The transfer literature adds a sharper version: observed moves were *chosen* by people forecasting the same quantity.
6. **Price is absent everywhere.** No source models fee or wages, so none of this yields value for money.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist a market; [[scoutgpt|counterfactual simulation]] to assess fit at a specific club; [[player-development-curve|PDC]] position to distinguish appreciating from depreciating assets.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP against O-VAEP]].
- **Assessing squad risk rather than mean output** → [[performance-volatility|volatility metrics]], residualised against rating.
- **Evaluating aerial or physical targets** → [[duel-skill-rating]], the only framework in the vault that rates them properly.
- **Analysing passages of play** → VAEP's context sensitivity.
- **Team-level possession quality** → [[lpv]] (interpretable units) or [[hpus]] (if timing matters).
- **Opponent scouting / training design** → [[football-event-sequences-point-process-mixture|possession clustering]].
- **Forecasting the next action** → Sig-Model if location matters, NMSTPP if timing does.
- **Tactical and off-ball analysis with tracking data** → martingale-EPV-style models.
- **Shot quality alone** → xG, a *component* of xT, VAEP, LPV and PCR rather than a competitor.

## See Also

- [[action-valuation]] · [[expected-possession-value]] · [[counterfactual-simulation]]
- [[expected-threat]] · [[vaep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]]
- [[large-event-model]] · [[mixture-model]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]] · [[intent-vs-outcome-valuation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[feature-engineering]] · [[selection-bias]] · [[positive-unlabeled-learning]]
