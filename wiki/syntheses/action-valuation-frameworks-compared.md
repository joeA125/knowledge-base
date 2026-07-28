---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, clustering, event-prediction, reliability, predictive-validity, time-series, volatility, player-development, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
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

Note that forecasting has quietly acquired a second output type. [[trajectory-prediction]] forecasts continuous positions rather than discrete events, and its main use here is not forecasting at all but supplying a [[counterfactual-baseline|reference]] for valuation.

## Task 1: Valuation

All frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

differing in what $S$ contains and how $Q$ is computed.

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Mart. EPV]] | [[pass-carry-reward\|Shelopugin]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[c-obso]] |
|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | Attack | Attack | **Defence** | Attack |
| **Whose value** | Actor | Actor | Actor | Actor | Actor | Team | **A teammate's** |
| **Data** | Event | Event | Tracking | Event | Tracking | Event+tracking | Tracking |
| **Target frequency** | Rare | **Very rare** | Rare | Dense (xG) | Rare | **Frequent** | Rare |
| **On/off ball** | On | On | On | On | **Both** | **Off (team)** | **Off** |
| **Mechanism** | DP | Boosting | Bayesian process | Boosting ×9 | Neural ×9 | XGBoost ×2 | **Counterfactual** |
| **[[interpretability]]** | **High** | Low | Low | Low | Moderate | Moderate | Moderate |
| **[[split-half-reliability\|Reliability]]** | **0.89** | 0.25 | — | — | N/A | N/A | — |
| **Output unit** | Player | Player | Player | Player | Situation | **Team** | Player |

**The central trade-off:** richer state buys sensitivity and pays in stability, interpretability and cost. Restricting VAEP to xT's action set only lifts reliability from 0.25 to 0.59, so the richer *representation* itself introduces variance.

**Four of seven produce no usable player season rating** — Fernández values situations, VDEP aggregates to teams, martingale EPV needs EPVA to escape its zero-mean property, and C-OBSO is defined only on shot-ending sequences. The frameworks most often called state-of-the-art are the least usable for [[recruitment]].

### Axis 1: Perspective — attacking or defending

Every framework except [[vdep]] measures attacking success and treats defence as its negative. [[football-defence-evaluation-vdep|Toda et al.]] measure the cost: **VAEP's conceding classifier scores F1 = 0.000** on 45 matches — no true positives at all, having learned to predict "no goal" always (right 99.2% of the time). The defensive half of the vault's most-cited framework is empirically inert at that data scale.

Offensive bias therefore has **four causes with four remedies**:

| Cause | Remedy | Status |
|---|---|---|
| Definitional — value is proximity to scoring | Change the target | [[vdep]] |
| Data — event streams cannot judge tackles | [[optical-tracking-data\|Tracking]] | Partial |
| Modelling choice — duel information exists, unmodelled | Model those events | [[duel-skill-rating]] |
| **Statistical — 227 positives cannot train a classifier** | **Frequent proxy** | **[[vdep]]** |

Van Dijk ranking 81st by VAEP and 142nd by xT while topping *both* of Shelopugin's duel tables is the clearest illustration that this is not one problem.

### Axis 2: Target rarity

See [[rare-event-proxy-targets]]. [[vdep]] swaps goals conceded for ball recovery and effective attack (~90× and ~35× more frequent); [[pass-carry-reward|Shelopugin]] swaps binary goal for accumulated xG; [[hpus]] uses **no goal data at all** and still correlates 0.92 with season xG.

The cost: **the proxy becomes the definition.** VDEP measures recovery-and-penetration performance, which is a *hypothesis about* defensive quality.

### Axis 3: Whose value — actor or beneficiary

New, and the vault's newest capability. Every framework above except [[c-obso]] credits the player **performing** the valued act. C-OBSO credits the player whose movement improved *someone else's* chance:

$$V_i = V^k_{OBSO} - V'^k_{OBSO}$$

This is relational credit, and no other framework here expresses it. It is why C-OBSO correlates with salary (ρ = 0.45) on players where [[obso|OBSO]] (−0.28) and goals (−0.23) do not — the space a player makes for others tracks what clubs pay him; his own opportunities and finishing do not.

### Axis 4: Intent vs outcome

Whether the model sees how the action turned out. xT and xG are effectively pure intent; VAEP conflates; I-VAEP/O-VAEP separates. Agüero's VAEP/xT gap is not a dispute about quality but xT measuring intent and VAEP measuring intent-plus-execution. See [[intent-vs-outcome-valuation]].

### Axis 5: Attributable possession

Undefined for an aerial duel, where two opposing players contest a ball neither holds. Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it.

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

**Five frameworks now carry an unjustified free parameter** — $\gamma = 0.95$, $\epsilon = 15$s, $k = 5$, $C = 3.9$, 4 s — and **not one reports a sensitivity analysis.** The clearest shared methodological weakness in this literature.

## Off-Ball Valuation: Three Mechanisms

Now substantial enough to treat separately. A player has the ball for roughly **3 of 90 minutes**; everything else was invisible to this vault until recently.

| | Surface at position | 22 positions in state | Predicted reference |
|---|---|---|---|
| Values | The **receiver** | The **defence** | The **creator** |
| Output unit | Player | **Team only** | Player |
| Example | EPV surface, [[obso]] | [[vdep]] | [[c-obso]] |

**The individuating ingredient is the counterfactual, not the data.** VDEP and C-OBSO use comparable tracking data. VDEP puts everything into one classifier and gets one number per configuration with no principled way to split it; C-OBSO intervenes on one *named* player and gets his number. Wherever collective data resists per-agent attribution, a counterfactual on one agent is the route out. See [[counterfactual-baseline]].

That pattern — evaluate by deviation from a predicted reference — also underlies EPVA (league-average player), Umemoto & Fujii's defensive positioning (best alternative cell), and Fernández's realised-vs-available gap. The **reference type** changes the question: deviating from expectation is not exceeding the average, and neither is approaching optimal.

Its weakness is shared too. [[c-obso]] is **identically zero under perfect prediction** — the metric requires its own reference model to be wrong, so values are not portable across predictors and improving the predictor shrinks the signal.

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Predicts | Next event | Event + time | Event + exact $(x,y)$ | Event + lineup | **Positions of all agents** |
| Encoder | [[transformer]] | Transformer | [[path-signature]] | GPT-2 | **[[graph-neural-network\|GNN]] + VAE** |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

The [[feature-engineering]] row records a finding that generalises: Seq2Event degrades without handcrafted geometry, Sig-Model degrades *with* it.

**A genuine tension.** Fernández et al. hand-engineer extensively ([[pitch-control]], [[dynamic-pressure-lines]]); VDEP hand-engineers a sorted off-ball state that [[shap]] confirms matters. By Sig-Model's finding this should hurt. The resolution is that they optimise different things — accuracy versus communicability — but neither side tests the other's claim.

GVRNN's margin over its own ablation is the largest in the vault: 0.608 m endpoint error against VRNN's 5.952 m at 4 seconds. Nearly an order of magnitude from adding relational structure — though the comparison also confounds centralised against per-player optimisation.

## Task 3: Clustering

[[football-event-sequences-point-process-mixture|Amezouwui et al.]] cluster whole possessions into tactical types via a [[mixture-model]] of [[point-process|marked spatio-temporal point processes]]. No outcome labels, no notion of value. Validated by [[adjusted-rand-index|ARI]] on simulated data, BIC, and interpretability.

## Task 4: Counterfactual and Transfer

| | [[scoutgpt\|Generative simulation]] | [[transfer-performance-prediction\|Regression on context]] |
|---|---|---|
| Destination encoded as | Explicit lineup | [[league-strength-rating\|Club/league strength]] |
| Captures tactical interaction | **Yes** | No |
| Handles role change | Implicitly | **No** |
| Scales to a market | No | **Yes** |
| Addresses [[selection-bias\|selection]] | Not addressed | Explicitly, heuristically |

**Regression to narrow a market, simulation to discriminate among survivors.** See [[recruitment]].

Distinguish this from [[counterfactual-baseline]]: simulation substitutes an *entity*; a baseline substitutes *predicted behaviour* for the same entity. Simulation answers recruitment questions, baselines answer valuation questions.

## Task 5: Tactical Analysis

The task coaches ask about most and analytics has served worst, because a tactic is a *relationship among positions over time* and event streams discard exactly that.

- **Value surfaces conditioned on shape** — [[expected-value-possession-framework|Fernández et al.]], concluding 4-3-3 against Liverpool unless wing-backs can press wide receptions.
- **Possession clustering** — what a team does, with no notion of value.
- **Defensive style profiling** — [[vdep]] plots recovery rate against being-attacked rate, separating high-press-high-risk from solid-and-contained.

**Weakest validation in the vault, structurally.** "4-3-3 is the better press" is a causal counterfactual about a formation that was not used, on observational data where formations were chosen for reasons. Worth flagging because tactical output is unusually *persuasive* — heatmaps read as evidence in a way a correlation table does not.

## Time as a Cross-Cutting Axis

Every task produces a number that is implicitly an average over a period. [[player-rating-time-series|Treating it as a series]] yields form, [[performance-volatility|volatility]], style change, and [[player-development-curve|career trajectory]] — none of which an average can express.

**An unresolved conflict.** [[split-half-reliability]] treats within-season variation as noise and marks VAEP down; volatility analysis treats it as signal. Both cannot be wholly right, and the decisive experiment is unrun.

Underneath sits a denominator question: per-90 assumes clock minutes measure opportunity, and [[effective-playing-time|effective playing time]] varies by team, scoreline and league. Only Shelopugin normalises on it.

## Metrics Beat Outcomes at Predicting Outcomes

| | poss-util | [[hpus]] | [[lpv]] | xG | goals |
|---|---|---|---|---|---|
| vs next-match xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs next-match goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

**Goals are the worst predictor of future goals.** A scoreline is a small, noisy sample of an underlying process. See [[predictive-validity]].

**Player-level.** [[epv-control-duel-skills-football|Shelopugin]] predicts next-season PCR against persistence: RMSE 0.053 → 0.033, and 0.061 → 0.037 for players changing both club and league. But it predicts *the metric's own future value* — self-prediction shows persistence, not validity.

**Cross-horizon consistency.** [[vdep]]'s match- and season-level correlations are similar (0.464, 0.397) while VAEP's diverge sharply (0.830 → 0.177). A metric that tracks the match it measures but not the season is reproducing the scoreline. This check is nearly free and nobody else reports it.

**External-criterion validation.** [[c-obso]] against annual salary is the vault's only attempt to validate against a measure originating entirely outside the modelling pipeline. Heavily confounded — by age, position, nationality, contract timing — but structurally the strongest available test, and the comparison against OBSO and goals on the same players is what carries it.

## How Each Task Is Validated

| Task | Validation |
|---|---|
| Valuation | Concurrent correlation; [[split-half-reliability]]; [[predictive-validity]]; [[probability-calibration\|calibration]]; [[class-imbalance-evaluation\|F1 under imbalance]]; external criteria (salary, expert ratings) |
| Forecasting | Held-out likelihood, Brier, [[kl-divergence]]; endpoint error for trajectories |
| Clustering | [[adjusted-rand-index]] on simulated data; BIC; interpretability |
| Counterfactual | Self-to-self reconstruction; out-of-sample transfer prediction |
| Transfer regression | RMSE against persistence, **stratified by whether the player moved** |
| Tactical | *None — illustrative only* |

Calibration and F1-under-imbalance each come from a single source. They are orthogonal and both needed: a model predicting the base rate every time is *perfectly calibrated* and finds nothing. Under football's imbalance, Brier and AUC are inflated by true negatives — **VAEP scores better than VDEP on Brier while scoring 0.000 on F1.**

**Nothing here has ever been benchmarked against anything else on a shared task.** VDEP is the closest attempt and is not like-for-like. This remains the field's largest methodological gap, and is why the tables above compare design characteristics rather than results.

## A Terminology Warning

"Expected possession value" means at least **four** things. See [[expected-possession-value]].

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies, as above.
2. **On-ball bias, now substantially narrowed.** Three mechanisms cover receiver value, team defensive contribution, and space creation for teammates. Still uncovered: **errors of omission** (a defender who fails to press generates no event), and movement over time beyond short windows.
3. **Individual defensive credit** — *not* held here, but addressed in the literature by Umemoto & Fujii (2023) via counterfactual positioning. See [[defensive-valuation]]. *(Corrected 2026-07-27; previously stated as completely open.)*
4. **No ground truth**, which is why reliability, predictive validity, calibration, imbalance-robust metrics and external criteria have all become substitute tests — and why self-prediction keeps getting mistaken for validation.
5. **Context dependence.** Value accumulates more easily in a weaker league or stronger team.
6. **[[selection-bias]] throughout** — minutes thresholds; transfers chosen by people forecasting the same quantity; [[single-pixel-supervision|pass surfaces]] best-constrained where passes are already plausible.
7. **Scale limits on interaction models.** [[c-obso]] predicts 3 of 22 players for computational reasons; the full-squad version is described as prohibitively expensive. A method whose appeal is modelling interaction is restricted to the smallest interacting subset.
8. **Two uncompared pitch-control traditions** — Spearman's arrival-time Poisson model and Fernández & Bornn's Gaussian influence model — feeding different value models, so a difference propagates silently.
9. **Price is absent everywhere.**
10. **No cross-framework benchmarking.**

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist; [[scoutgpt|simulation]] for fit; [[player-development-curve|PDC]] for trajectory.
- **Valuing a player who does not score** → [[c-obso]], the only metric here shown to track an external measure of worth where goals do not.
- **Assessing a defence** → [[vdep]] (team level only).
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]], the only real-time framework.
- **Valuing what was available rather than what happened** → [[probability-surface|pass surfaces]].
- **Off-ball and positional analysis** → surfaces plus [[pitch-control]]; C-OBSO for space creation.
- **Opposition and pressing analysis** → [[tactical-analysis]], with the validation caveat.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Squad risk rather than mean output** → [[performance-volatility|volatility]], residualised against rating.
- **Aerial or physical targets** → [[duel-skill-rating]].
- **Team-level possession quality** → [[lpv]] or [[hpus]].
- **Shot quality alone** → xG, a *component* of the others rather than a competitor.

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[obso]] · [[c-obso]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[graph-neural-network]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[pitch-control]] · [[dynamic-pressure-lines]]
- [[structured-model-decomposition]] · [[policy-modelling]] · [[interpretability]] · [[shap]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[eventgpt]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]] · [[intent-vs-outcome-valuation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[feature-engineering]] · [[selection-bias]] · [[positive-unlabeled-learning]]
- [[william-spearman]] · [[keisuke-fujii]]
