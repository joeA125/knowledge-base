---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, player-evaluation, evaluation, counterfactual, clustering, event-prediction, reliability, predictive-validity, time-series, volatility, player-development, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, off-ball, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md]
confidence: 0.9
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **five distinct tasks**, each answering a different question, with different data requirements and different validation strategies.

*(Formerly "Action Valuation Frameworks Compared" — renamed as the vault outgrew the valuation-only framing.)*

## The Five Tasks

| Task | Question | Unit | Needs outcome labels? | Examples |
|---|---|---|---|---|
| **Valuation** | How good was that action? | Action → player or team | Yes | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event | No | [[seq2event]], [[nmstpp]], [[sig-model]] |
| **Clustering** | What kind of sequence is this? | Possession | No | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | No (uses value as target) | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | No | [[tactical-analysis\|Pressing analysis]], possession clustering |

**Defensive valuation is not a sixth task.** [[vdep]] instantiates the same equation as every other valuation framework — it differs in *perspective* (whose success is measured) and in *target choice* (what proxy stands in for the outcome). Both are axes within Task 1, developed below. Treating defence as its own task would obscure that the machinery is identical and only the sign and the labels change.

**A cross-cutting dimension.** [[football-performance-time-series|Mendes-Neves et al.]] attack the **aggregation step** shared by all five tasks. Every one ends by collapsing something over time, treated as a formality everywhere except in their work. See [Time as a Cross-Cutting Axis](#time-as-a-cross-cutting-axis).

## Task 1: Valuation

All valuation frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

differing only in what $S$ contains and how $Q$ is computed.

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Martingale EPV]] | [[pass-carry-reward\|Shelopugin]] | [[expected-value-possession-framework\|Fernández et al.]] | [[vdep]] |
|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | Attack | Attack | **Defence** |
| **Data** | [[event-stream-data\|Event]] | Event | [[optical-tracking-data\|Tracking]] | Event | Tracking | Event **+ tracking** |
| **Target** | P(goal this poss.) | P(score)−P(concede) | Points, martingale | Decayed future xG | Which team scores next | **Recovery / being attacked** |
| **Target frequency** | Rare | **Very rare** | Rare | Dense (xG) | Rare | **Frequent** |
| **State $S$** | Ball's zone | Last 3 actions | Full tracking history | Action + duel skill | Full tracking snapshot | Action **+ all 22 positions** |
| **Estimation** | [[value-iteration]] | [[gradient-boosting]] | Bayesian [[multiresolution-modelling\|multiresolution]] | Boosting ×9 | 9 neural, [[structured-model-decomposition\|decomposed]] | XGBoost ×2 |
| **Models risk** | No | Yes (10 actions) | Implicitly | Yes (unbounded) | Natively | **Is the risk model** |
| **Values duels** | No | No | No | **Yes** | No | No |
| **Values off-ball** | No | No | Partially | No | **Yes (attacking)** | **Yes (defending)** |
| **[[interpretability]]** | **High** | Low | Low | Low | Moderate (compositional) | Moderate ([[shap]]) |
| **[[split-half-reliability\|Reliability]]** | **ρ = 0.89** | ρ = 0.25 | Not reported | Not reported | N/A — no player rating | N/A — **team only** |
| **Output unit** | Player | Player | Player | Player | Situation | **Team** |

**The central trade-off:** richer state buys sensitivity and pays in stability, interpretability, and cost. [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] show the reliability gap is not merely about scope — restricting VAEP to xT's action set only recovers $\rho = 0.25 \to 0.59$.

Note that **three of the six produce no player rating at all**, for three different reasons: Fernández et al. value situations, VDEP cannot individuate defensive credit, and martingale EPV needs the EPVA construction to escape its own zero-mean property. The frameworks most often described as state-of-the-art are the least usable for [[recruitment]].

### Axis 1: Perspective — attacking or defending

Every framework except [[vdep]] measures **attacking** success and treats defence as its negative: VAEP's $P_{concedes}$, xT's absence of a conceding term, PCR's opponent-side subtraction.

[[football-defence-evaluation-vdep|Toda et al.]] measure what that assumption costs. On a 45-match dataset, **VAEP's conceding classifier achieves F1 = 0.000** — it identifies no true positives whatsoever, having learned to predict "no goal" always, which is right 99.2% of the time. The defensive half of the vault's most-cited valuation framework is not weak but *empirically inert* at this data scale. It is coherent with something the vault had already recorded without explanation: VAEP correlates ≈0 with goals conceded ($r = -0.098$ season) despite being built from a conceding model.

This reframes the offensive-bias problem that appears on every page here. It has **four causes with four different remedies**:

| Cause | Remedy | Status |
|---|---|---|
| Definitional — value is proximity to scoring | Change the target | [[vdep]] |
| Data — event streams cannot judge tackles | [[optical-tracking-data\|Tracking]] | Partial |
| Modelling choice — duel information exists, unmodelled | Model those events | [[duel-skill-rating]] |
| **Statistical — 227 positives cannot train a classifier** | **Frequent proxy** | **[[vdep]]** |

The fourth is new, and is the one nobody had measured. Van Dijk ranking 81st by VAEP and 142nd by xT while topping *both* of Shelopugin's duel tables is the clearest single illustration that this is not one problem.

### Axis 2: Target rarity

A methodological choice usually presented as a domain trick, and general enough to deserve its own name — see [[rare-event-proxy-targets]].

| Framework | Target abandoned | Proxy adopted |
|---|---|---|
| [[vdep]] | Goals conceded | Ball recovery, effective attack (~90×, ~35× more frequent) |
| [[pass-carry-reward\|Shelopugin]] | Binary "possession ended in goal" | Accumulated future xG |
| [[hpus]] | Goals | Possession utilisation — **no goal data at all** |

The strongest evidence that the move works is [[hpus]], which uses no goal or shot-outcome data at any stage yet correlates 0.92 with season xG and −0.78 with league position, against xG's −0.81.

The cost is that **the proxy becomes the definition**. VDEP does not measure defensive quality; it measures recovery-and-penetration performance, which is a *hypothesis about* defensive quality.

### Axis 3: Intent vs outcome

Whether the model can see how the action turned out. [[expected-threat|xT]] and [[expected-goals|xG]] are effectively pure intent; [[vaep]] conflates; I-VAEP/O-VAEP separates explicitly. Several apparent disagreements dissolve here — Agüero's VAEP/xT gap is not a dispute about quality but xT measuring intent and VAEP measuring intent-plus-execution. See [[intent-vs-outcome-valuation]].

### Axis 4: Attributable possession

The unifying equation needs to know *whose* prospects $Q$ describes — undefined for an aerial duel. Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it; the [[epv-control-duel-skills-football|Donnarumma case]] values two tactically identical long passes at 0.00075 and 0.00077 duel-blind, separated roughly two-fold duel-aware.

### Axis 5: Realised vs available

Every framework except Fernández et al. values only what happened. A full [[probability-surface|pass surface]] values every option, making the gap between realised (0.032) and best-available (0.112) computable. **That gap is the coaching output.** See [[policy-modelling]].

### Credit assignment over time

| Approach | Boundary | Framework |
|---|---|---|
| Whole possession, undecayed | Turnover | Early EPV / xT |
| Fixed $k$-action window | Action count ($k = 10$) | [[vaep]] |
| Capped time decay | 1 min, floored at 5 actions | [[football-performance-time-series\|Mendes-Neves et al.]] |
| Hard time cutoff | $\epsilon = 15$s, then zero | [[expected-value-possession-framework\|Fernández et al.]] |
| Fixed event window | $k = 5$ events | [[vdep]] |
| Geometric time decay | None — weight → 0 | [[temporal-discounting\|Shelopugin]] |

Direction of travel is from counting actions to measuring elapsed time; the endpoint dissolves the horizon question entirely, since distant events contribute nothing.

**Four frameworks now carry an unjustified free parameter** — $\gamma = 0.95$, $\epsilon = 15$s, $k = 5$, $C = 3.9$ — and **not one reports a sensitivity analysis.** Since $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$, rankings are almost certainly sensitive to at least the first. This is the clearest shared methodological weakness in the literature held here.

## Task 2: Forecasting

| | [[seq2event]] (2022) | [[nmstpp]] (2023) | [[sig-model]] (2025) | [[scoutgpt]] (2026) |
|---|---|---|---|---|
| History | Fixed window | Fixed window (40) | **Whole possession** | Episode + lineup |
| Encoder | [[transformer]] | Transformer | [[path-signature]] + FFN | GPT-2 decoder |
| Forecasts time? | No | **Yes** | No | Yes |
| Location | Zone | Zone (20) | **Exact $(x,y)$** | Binned $(x,y)$ |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP |

Each step objects to something in the last: NMSTPP says Seq2Event ignores *when*; Sig-Model says both use the wrong *unit*; ScoutGPT says none can be *conditioned on a hypothetical lineup*.

The [[feature-engineering]] row records a finding that generalises: Seq2Event degrades without handcrafted geometry, Sig-Model degrades *with* it.

**A genuine tension with the tracking line.** Fernández et al. hand-engineer extensively ([[pitch-control]], [[dynamic-pressure-lines]]) with club analysts; VDEP hand-engineers a sorted off-ball state whose value [[shap]] confirms. By Sig-Model's finding this should hurt. The resolution is that they optimise different things — accuracy versus communicability — but neither side tests the other's claim.

## Task 3: Clustering

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] fit a [[mixture-model]] whose components are [[point-process|marked spatio-temporal point processes]], clustering whole possessions into tactical types. Needs no outcome labels and no notion of value. Validated by [[adjusted-rand-index|ARI]] on simulated data, BIC, and interpretability against tactical vocabulary.

## Task 4: Counterfactual and Transfer

| | [[scoutgpt\|Generative simulation]] | [[transfer-performance-prediction\|Regression on context]] |
|---|---|---|
| Object modelled | Event sequence | Season aggregate |
| Destination encoded as | Explicit lineup | [[league-strength-rating\|Club/league strength]] |
| Captures tactical interaction | **Yes** | No |
| Handles role change | Implicitly | **No** — acknowledged failure |
| Scales to a market | No | **Yes** |
| Addresses [[selection-bias\|selection]] | Not addressed | Explicitly, heuristically |

**Regression to narrow a market, simulation to discriminate among survivors.** See [[recruitment]].

The generative papers do not address that observed transfers were **chosen** by clubs forecasting the same quantity — a [[positive-unlabeled-learning|presence-only]] structure no destination conditioning corrects.

A third route is unexplored: [[probability-surface|pass surfaces]] make cheap counterfactuals available — the value of every unrealised option — without generation or Monte Carlo.

## Task 5: Tactical Analysis

The task coaches ask about most and analytics has served worst, because a tactic is a *relationship among positions over time* and event streams discard exactly that.

- **Value surfaces conditioned on shape** — [[expected-value-possession-framework|Fernández et al.]] compute off-ball and on-ball EPV heatmaps per opponent formation. The Liverpool analysis concludes 4-3-3 unless you have wing-backs quick enough to press wide receptions.
- **Possession clustering** — [[football-event-sequences-point-process-mixture|Amezouwui et al.]] characterise what a team does, with no notion of value.
- **Defensive style profiling** — [[vdep]] plots recovery rate against being-attacked rate, separating high-press-high-risk (Yokohama) from solid-and-contained (Hiroshima). Cheaper than either of the above and available from event data.

**This task has the weakest validation in the vault, structurally.** "4-3-3 is the better press" is a causal counterfactual about a formation that was not used, on observational data where formations were chosen for reasons. Worth flagging because tactical output is unusually *persuasive*: heatmaps read as evidence in a way a correlation table does not.

## Time as a Cross-Cutting Axis

Every task produces a number that is implicitly **an average over a period**. [[player-rating-time-series|Treating that period as a series]] yields quantities none of the tasks produce:

| Quantity | What it captures | Why an average cannot |
|---|---|---|
| Short- vs long-term rating | Form against underlying quality | Averaging conflates them by construction |
| [[performance-volatility\|Volatility]] | Week-to-week reliability | Two players, one mean, different risk |
| Per-action-type trajectory | Style *change* | Messi's dribble and pass value moved oppositely over a decade |
| [[player-development-curve\|Development curve]] | Career stage and peak | A 23- and a 30-year-old at equal output are different assets |

**An unresolved conflict with the reliability critique.** [[split-half-reliability]] treats within-season variation as noise and marks VAEP down for it; [[performance-volatility|volatility analysis]] treats the same variation as signal. Both cannot be wholly right. The decisive experiment is unrun.

A denominator question runs underneath: per-90 assumes clock minutes measure opportunity, and [[effective-playing-time|effective playing time]] varies by team, scoreline and league. Only Shelopugin normalises on it.

## Metrics Beat Outcomes at Predicting Outcomes

| | poss-util | [[hpus]] | [[lpv]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

**Goals are the worst predictor of future goals.** A scoreline is a small, noisy sample of an underlying process. See [[predictive-validity]].

### Player-level, and team-stability

[[epv-control-duel-skills-football|Shelopugin]] gives the first player-level evidence: next-season PCR against persistence, RMSE 0.053 → 0.033, and 0.061 → 0.037 for players changing both club and league. Persistence degrades monotonically with movement while the model holds. But it predicts **the metric's own future value**, not an independent outcome — self-prediction shows persistence, not validity.

[[vdep]] adds a different stability result. Its match-level and season-level correlations are similar (0.464, 0.397) while VAEP's diverge sharply (0.830 → 0.177). A metric that tracks the match it measures and not the season is reproducing the scoreline; one whose correlations hold across horizons is measuring a team property. **Consistency across time horizons is a validation check nobody else here reports**, and it is nearly free.

## How Each Task Is Validated

| Task | Validation |
|---|---|
| Valuation | Concurrent correlation; [[split-half-reliability]]; [[predictive-validity]]; [[probability-calibration\|calibration]]; **[[class-imbalance-evaluation\|F1 under imbalance]]** |
| Forecasting | Held-out likelihood, Brier, [[kl-divergence]] against empirical distributions |
| Clustering | [[adjusted-rand-index]] on simulated data; BIC; interpretability |
| Counterfactual | Self-to-self reconstruction; out-of-sample transfer prediction |
| Transfer regression | RMSE/MAE against persistence, **stratified by whether the player moved** |
| Tactical | *None — illustrative only* |
| *Time-series derivatives* | *Weakest — face validity and agreement on peak age only* |

Two additions are recent and both come from single sources. **Calibration** (Fernández et al.) asks whether numbers mean what they say — essential for a decomposed model, since well-calibrated parts are what license trust in a recombined whole. **F1 under imbalance** (Toda et al.) asks whether the model finds the rare positives at all.

The two are orthogonal and both are needed: a model predicting the base rate every time is *perfectly calibrated* and finds nothing. Under football's class imbalance, Brier and AUC are inflated by true negatives — which is why **VAEP scores better than VDEP on Brier while scoring 0.000 on F1.** Reading that comparison by Brier alone reverses the correct conclusion. See [[class-imbalance-evaluation]].

**Nothing here has ever been benchmarked against anything else on a shared task.** VDEP is the closest attempt and is not like-for-like — $k=5$ against $k=10$, different target events, a dataset far smaller than VAEP's original, as its authors say. This remains the field's largest methodological gap, and it is why the comparison table above is assembled from design characteristics rather than results.

## A Terminology Warning

"Expected possession value" means at least **four** things: Cervone et al.'s basketball martingale; a soccer *category label* covering xT-style zonal models; Fernández, Bornn & Cervone's tracking framework; and Shelopugin's event-data model targeting accumulated xG. See [[expected-possession-value]].

## Limitations Shared Across Tasks

1. **Offensive bias** — four distinct causes, four remedies, as above. No longer a single paradigm limitation.
2. **On-ball only**, with two partial exceptions. [[off-ball-value|Fernández et al.]] value attacking off-ball *position*; [[vdep]] values defensive off-ball contribution at **team level**. Neither credits creating space for others, movement over time, or errors of omission.
3. **No ground truth**, which is why reliability, predictive validity, calibration and now imbalance-robust metrics have become substitute tests — and why self-prediction keeps getting mistaken for validation.
4. **Context dependence.** Value is easier to accumulate in a weaker league or stronger team.
5. **[[selection-bias]] throughout.** Minutes thresholds; transfers chosen by people forecasting the same quantity; [[single-pixel-supervision|pass surfaces]] best-constrained where passes are already plausible.
6. **Individual defensive credit is completely open.** VDEP is team-level by design. Its authors' proposed next step — compute the change in VDEP when a player moves differently — is unimplemented.
7. **Price is absent everywhere.**
8. **No cross-framework benchmarking**, as above.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist; [[scoutgpt|simulation]] for fit at a specific club; [[player-development-curve|PDC]] for trajectory.
- **Assessing a defence** → [[vdep]], the only framework that targets prevention directly. Team level only.
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]], the only real-time framework here.
- **Valuing what was available rather than what happened** → [[probability-surface|pass surfaces]].
- **Off-ball and positional analysis** → same, plus [[pitch-control]]; VDEP for the defensive side.
- **Opposition and pressing analysis** → [[tactical-analysis]], with the validation caveat.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP against O-VAEP]].
- **Squad risk rather than mean output** → [[performance-volatility|volatility]], residualised against rating.
- **Aerial or physical targets** → [[duel-skill-rating]].
- **Analysing passages of play** → VAEP's context sensitivity.
- **Team-level possession quality** → [[lpv]] or [[hpus]].
- **Forecasting the next action** → Sig-Model if location matters, NMSTPP if timing does.
- **Shot quality alone** → xG, a *component* of the others rather than a competitor.

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[expected-possession-value]] · [[counterfactual-simulation]] · [[tactical-analysis]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[off-ball-value]] · [[pitch-control]] · [[dynamic-pressure-lines]]
- [[structured-model-decomposition]] · [[policy-modelling]] · [[interpretability]] · [[shap]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[eventgpt]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]] · [[intent-vs-outcome-valuation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[feature-engineering]] · [[selection-bias]] · [[positive-unlabeled-learning]]
