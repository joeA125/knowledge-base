---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, clustering, event-prediction, reliability, predictive-validity, time-series, volatility, player-development, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md]
confidence: 0.9
provenance:
  extracted: 55%
  inferred: 42%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **five distinct tasks**, each answering a different question, with different data requirements and validation strategies.

*(Formerly "Action Valuation Frameworks Compared" — renamed as the vault outgrew the valuation-only framing.)*

## The Five Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[obso]], [[c-obso]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |

**Defensive valuation is not a sixth task.** [[vdep]] instantiates the same equation, differing in *perspective* and *target choice* — both axes within Task 1.

## Task 1: Valuation

All frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

differing in what $S$ contains and how $Q$ is computed.

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Mart. EPV]] | [[pass-carry-reward\|Shelopugin]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[obso]] | [[c-obso]] |
|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | Attack | Attack | **Defence** | Attack | Attack |
| **Whose value** | Actor | Actor | Actor | Actor | Actor | Team | Receiver | **A teammate's** |
| **On/off ball** | On | On | On | On | Both | Off (team) | **Off** | **Off** |
| **Data** | Event | Event | Tracking | Event | Tracking | Both | Tracking | Tracking |
| **Mechanism** | DP | Boosting | Bayesian process | Boosting ×9 | Neural ×9 | XGBoost ×2 | **Physical** | **Counterfactual** |
| **[[interpretability]]** | **High** | Low | Low | Low | Moderate | Moderate | **High** | Moderate |
| **[[split-half-reliability\|Reliability]]** | **0.89** | 0.25 | — | — | N/A | N/A | — | — |
| **Output unit** | Player | Player | Player | Player | Situation | **Team** | Player | Player |
| **Cost** | Trivial | Modest | **461 CPUs** | Modest | Modest | Modest | **Low** | High |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. Restricting VAEP to xT's action set only lifts reliability from 0.25 to 0.59, so the richer *representation* itself introduces variance.

[[obso|OBSO]] is the notable exception. It is tracking-based, off-ball, and highly interpretable, at low cost — because it is **physical rather than learned**. Arrival times come from acceleration limits, ball flight from aerodynamic drag, control from a Poisson process. Parameters have units, so priors come from measurement and five training matches suffice.

**Four of eight produce no usable player season rating** — Fernández values situations, VDEP aggregates to teams, martingale EPV needs EPVA to escape its zero-mean property, C-OBSO is defined only on shot-ending sequences.

### Axis 1: Perspective — attacking or defending

Every framework except [[vdep]] measures attacking success and treats defence as its negative. [[football-defence-evaluation-vdep|Toda et al.]] measure the cost: **VAEP's conceding classifier scores F1 = 0.000** on 45 matches — no true positives at all, having learned to predict "no goal" always (right 99.2% of the time).

Offensive bias has **four causes with four remedies**:

| Cause | Remedy | Status |
|---|---|---|
| Definitional — value is proximity to scoring | Change the target | [[vdep]] |
| Data — event streams cannot judge tackles | [[optical-tracking-data\|Tracking]] | Partial |
| Modelling choice — duel information exists, unmodelled | Model those events | [[duel-skill-rating]] |
| **Statistical — 227 positives cannot train a classifier** | **Frequent proxy** | **[[vdep]]** |

### Axis 2: Target rarity

See [[rare-event-proxy-targets]]. [[vdep]] swaps goals conceded for recovery and effective attack (~90× and ~35× more frequent); [[hpus]] uses **no goal data at all** and still correlates 0.92 with season xG. The cost: **the proxy becomes the definition.**

### Axis 3: Whose value — actor, receiver, or beneficiary

Three distinct answers now, where the vault long had one:

- **The actor** — every on-ball framework. Credits the player performing the valued act.
- **The receiver** — [[obso|OBSO]]. Credits the player whose *position* would be valuable if the ball arrived.
- **The beneficiary's creator** — [[c-obso]]. Credits the player whose movement improved *someone else's* chance.

The progression matters because each step credits someone the previous one could not see. C-OBSO correlates 0.45 with salary on players where OBSO (−0.28) and goals (−0.23) do not. See [[space-creation]].

### Axis 4: Intent vs outcome

Whether the model sees how the action turned out. xT and xG are effectively pure intent; VAEP conflates; I-VAEP/O-VAEP separates. See [[intent-vs-outcome-valuation]].

### Axis 5: Attributable possession

Undefined for an aerial duel. Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it.

### Axis 6: Realised vs available

A full [[probability-surface|pass surface]] values every option, making the gap between realised (0.032) and best-available (0.112) computable. **The gap is the coaching output.** See [[policy-modelling]].

### Credit assignment over time

| Approach | Boundary | Framework |
|---|---|---|
| Fixed $k$-action window | Action count ($k = 10$) | [[vaep]] |
| Capped time decay | 1 min, floored at 5 actions | [[football-performance-time-series\|Mendes-Neves et al.]] |
| Hard time cutoff | $\epsilon = 15$s | [[expected-value-possession-framework\|Fernández et al.]] |
| Fixed event window | $k = 5$ events | [[vdep]] |
| Fixed time window | 4 s prediction horizon | [[c-obso]] |
| Geometric time decay | None — weight → 0 | [[temporal-discounting\|Shelopugin]] |
| **Next on-ball event only** | **One step** | **[[obso]]** |

**Five frameworks carry an unjustified free parameter** — $\gamma$, $\epsilon$, $k$, $C$, 4 s — and none reports a sensitivity analysis. See [[model-selection]].

[[obso|Spearman]] is the exception, and instructive: all six of his parameters are MAP-fitted with stated priors and stated justifications, which is possible *because* they have physical units. Where a parameter means "seconds of temporal uncertainty", a prior can come from a previous measurement; where it means "stylistic preference for vertical attacking", it cannot.

## Off-Ball Valuation: Four Mechanisms

A player has the ball for roughly **3 of 90 minutes**. Until recently everything else was invisible here.

| | Surface at position | 22 positions in state | Predicted reference | Physical surface |
|---|---|---|---|---|
| Values | The receiver | The defence | The creator | The receiver |
| Output unit | Player | **Team only** | Player | Player |
| Example | EPV surface | [[vdep]] | [[c-obso]] | [[obso]] |

**The individuating ingredient is the counterfactual, not the data.** VDEP and C-OBSO use comparable tracking data; VDEP produces one number per configuration with no way to split it, C-OBSO intervenes on one *named* player. See [[counterfactual-baseline]].

That pattern's weakness is shared: [[c-obso]] is **identically zero under perfect prediction**, so the metric requires its own reference model to be wrong.

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Predicts | Next event | Event + time | Event + exact $(x,y)$ | Event + lineup | **All agents' positions** |
| Encoder | [[transformer]] | Transformer | [[path-signature]] | GPT-2 | **[[graph-neural-network\|GNN]] + VAE** |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

The [[feature-engineering]] row records a finding that generalises: Seq2Event degrades *without* handcrafted geometry, Sig-Model degrades *with* it. Engineered features are a crutch for a representation that cannot recover the geometry itself. See [[representation-learning]].

**Forecasting produces metrics as a by-product**, and those metrics need no outcome labels — so goal sparsity never bites. See [[event-prediction]].

## Task 3: Clustering

[[football-event-sequences-point-process-mixture|Amezouwui et al.]] cluster possessions into tactical types via a [[mixture-model]] of [[point-process|marked spatio-temporal point processes]]. Validated by [[adjusted-rand-index|ARI]] on simulated data, BIC, and interpretability. See [[clustering]].

## Task 4: Counterfactual and Transfer

| | [[scoutgpt\|Generative simulation]] | [[transfer-performance-prediction\|Regression on context]] |
|---|---|---|
| Destination encoded as | Explicit lineup | [[league-strength-rating\|Club/league strength]] |
| Captures tactical interaction | **Yes** | No |
| Scales to a market | No | **Yes** |
| Addresses [[selection-bias\|selection]] | Not addressed | Explicitly, heuristically |

**Regression to narrow a market, simulation to discriminate among survivors.** Distinguish from [[counterfactual-baseline]]: simulation substitutes an *entity*; a baseline substitutes *predicted behaviour* for the same entity.

## Task 5: Tactical Analysis

- **Value surfaces conditioned on shape** — [[expected-value-possession-framework|Fernández et al.]], concluding 4-3-3 against Liverpool unless wing-backs can press wide receptions.
- **Possession clustering** — what a team does, with no notion of value.
- **Defensive style profiling** — [[vdep]] separates high-press-high-risk from solid-and-contained.
- **Player-specific danger maps** — [[obso|OBSO]] maps stay stable across matches while shots and goals fluctuate, identifying where a specific opponent creates threat.

**Weakest validation in the vault, structurally.** These are causal counterfactuals about configurations that were not used, on observational data where they were chosen for reasons. Worth flagging because tactical output is unusually *persuasive* — heatmaps read as evidence in a way a correlation table does not.

## Time as a Cross-Cutting Axis

Every task produces a number implicitly averaged over a period. [[player-rating-time-series|Treating it as a series]] yields form, [[performance-volatility|volatility]], style change and [[player-development-curve|career trajectory]].

**An unresolved conflict.** [[split-half-reliability]] treats within-season variation as noise; volatility analysis treats it as signal. Both cannot be wholly right, and the decisive experiment is unrun.

Underneath sits a denominator question: per-90 assumes clock minutes measure opportunity, and [[effective-playing-time|effective playing time]] varies by team, scoreline and league.

## Metrics Beat Outcomes at Predicting Outcomes

The vault's most robust empirical finding, now established at **both levels by independent sources seven years apart.**

**Player level** ([[beyond-expected-goals|Spearman, 2018]]), match $i$ → match $i{+}1$ across 53 matches:

| Predictor | Next-match goals |
|---|---|
| **[[obso\|OBSO]]** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**Team level** ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]):

| | poss-util | [[hpus]] | [[lpv]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

**Goals are the worst predictor of future goals in both** — 0.12 and 0.11, arrived at independently. A scoreline is a small, noisy sample of an underlying process; a possession-value metric aggregates hundreds of actions and estimates that process directly. **Measuring the process beats measuring the outcome when the outcome is sparse.**

Spearman's is the stronger of the two, and the strongest result in the vault. It clears three bars simultaneously: **player-level**, against an **independent outcome**, and **beating the outcome's own lagged value**. He states the objective outright — a leading indicator less stochastic than scoring itself.

Contrast [[epv-control-duel-skills-football|Shelopugin]], whose next-season PCR forecast (RMSE 0.053 → 0.033) predicts **the metric's own future value**. Self-prediction shows persistence, not validity: a metric tracking tactical role rather than quality would score just as well.

[[vdep]] adds a third check nobody else reports — **cross-horizon consistency**. Its match- and season-level correlations are similar (0.464, 0.397) while VAEP's diverge sharply (0.830 → 0.177). A metric that tracks the match it measures but not the season is reproducing the scoreline.

And [[c-obso]] adds a fourth — **external criterion**. Against annual salary, C-OBSO correlates 0.45 where OBSO and goals do not. Heavily confounded, but the only validation here against a measure originating entirely outside the modelling pipeline.

## How Each Task Is Validated

| Task | Validation |
|---|---|
| Valuation | Concurrent correlation; [[split-half-reliability]]; [[predictive-validity]]; [[probability-calibration\|calibration]]; [[class-imbalance-evaluation\|F1 under imbalance]]; external criteria |
| Forecasting | Held-out likelihood, Brier, [[kl-divergence]]; endpoint error for trajectories |
| Clustering | [[adjusted-rand-index]] on simulated data; BIC; interpretability |
| Counterfactual | Self-to-self reconstruction; out-of-sample transfer prediction |
| Transfer regression | RMSE against persistence, **stratified by whether the player moved** |
| Tactical | *None — illustrative only* |

Calibration and F1-under-imbalance are orthogonal and both needed: a model predicting the base rate every time is *perfectly calibrated* and finds nothing. **VAEP scores better than VDEP on Brier while scoring 0.000 on F1.**

**Nothing here has ever been benchmarked against anything else on a shared task.** VDEP is the closest attempt and is not like-for-like; Spearman compares against nothing at all. This remains the field's largest methodological gap, and is why the tables above compare design characteristics rather than results.

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed.** Four mechanisms now cover receiver value, team defensive contribution, space creation, and physical off-ball opportunity. Still uncovered: **errors of omission**, and movement over time beyond short windows.
3. **Individual defensive credit** — addressed in the literature (Umemoto & Fujii, 2023) but not held here.
4. **No ground truth**, which is why reliability, predictive validity, calibration, imbalance-robust metrics and external criteria have all become substitute tests.
5. **Context dependence.** Value accumulates more easily in a weaker league or stronger team.
6. **[[selection-bias]] throughout** — minutes thresholds; transfers chosen by people forecasting the same quantity; [[single-pixel-supervision|pass surfaces]] best-constrained where passes are already plausible.
7. **Scale limits on interaction models.** [[c-obso]] predicts 3 of 22 players; full-squad is prohibitively expensive.
8. **Two uncompared [[pitch-control]] traditions** — Spearman's arrival-time Poisson model and Fernández & Bornn's Gaussian influence model. They differ on **additivity** (summed Gaussians over-count overlapping coverage) and on **ball travel time** (modelled with drag in one, instantaneous in the other), so they are least likely to agree in congested areas near goal. Both feed value models whose outputs *are* compared.
9. **Price is absent everywhere.**
10. **No cross-framework benchmarking.**

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist; [[scoutgpt|simulation]] for fit; [[player-development-curve|PDC]] for trajectory.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] for positioning, [[c-obso]] for space created. The two metrics here with external validation.
- **Assessing a defence** → [[vdep]] (team level only).
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]]; [[obso|OBSO]] for ranking moments for video review at far lower cost.
- **Valuing what was available rather than what happened** → [[probability-surface|pass surfaces]].
- **Opposition and pressing analysis** → [[tactical-analysis]], with the validation caveat.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Squad risk rather than mean output** → [[performance-volatility|volatility]], residualised against rating.
- **Aerial or physical targets** → [[duel-skill-rating]].
- **Team-level possession quality** → [[lpv]] or [[hpus]].
- **Shot quality alone** → xG, a *component* of the others rather than a competitor.

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[obso]] · [[c-obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[dynamic-pressure-lines]]
- [[structured-model-decomposition]] · [[policy-modelling]] · [[interpretability]] · [[representation-learning]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[eventgpt]] · [[event-prediction]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]] · [[intent-vs-outcome-valuation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[positive-unlabeled-learning]]
- [[william-spearman]] · [[keisuke-fujii]] · [[javier-fernandez]]
