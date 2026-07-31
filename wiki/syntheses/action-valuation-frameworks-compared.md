---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **six distinct tasks**, each answering a different question, with different data requirements and validation strategies.

*(Formerly "Action Valuation Frameworks Compared" — renamed as the vault outgrew the valuation-only framing.)*

> **Eight open questions have their own pages**, listed under Questions in the index. Where this synthesis flags an unresolved problem, the question page carries the analysis. Pointers are given inline below.

## The Six Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[obso]], [[c-obso]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision | [[xsot\|Yeung & Fujii]] |

**Prescription is genuinely a sixth task.** Every other framework describes what happened and assigns it a value. [[game-theory|Yeung & Fujii]] solve for what *ought* to happen, and can defend the claim by pointing at a payoff table. That is a different epistemic object, not a variation on valuation.

## Task 1: Valuation

All frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Mart. EPV]] | [[pass-carry-reward\|Shelopugin]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[obso]] | [[c-obso]] | [[xsot\|xSOT]] |
|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | Attack | Attack | **Defence** | Attack | Attack | **Both** |
| **Whose value** | Actor | Actor | Actor | Actor | Actor | Team | Receiver | **Teammate's** | Actor |
| **Target** | Goal | Goal | Points | Decayed xG | Next goal | **Recovery** | Goal | Goal | **Shot on target** |
| **On/off ball** | On | On | On | On | Both | Off (team) | **Off** | **Off** | Both |
| **Mechanism** | DP | Boosting | Bayesian process | Boosting ×9 | Neural ×9 | XGBoost ×2 | **Physical** | **Counterfactual** | **[[theory-based-modelling\|Hybrid]] + game** |
| **[[interpretability]]** | **High** | Low | Low | Low | Moderate | Moderate | **High** | Moderate | Moderate |
| **[[split-half-reliability\|Reliability]]** | **0.89** | 0.25 | — | — | N/A | N/A | — | — | — |
| **Cost** | Trivial | Modest | **461 CPUs** | Modest | Modest | Modest | **Low** | High | Low |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]] is the exception: tracking-based, off-ball, highly interpretable, at low cost, because it is **physical rather than learned**.

**Five of nine produce no usable player season rating.** The frameworks most often called state-of-the-art are the least usable for [[recruitment]].

### Axis 1: Perspective — attacking or defending

[[football-defence-evaluation-vdep|Toda et al.]] measure the cost of the attacking default: VAEP's conceding classifier scores **F1 = 0.000** on 45 matches. Offensive bias has **four causes with four remedies** — definitional (change the target), data (tracking), modelling choice (model duels), and statistical (a frequent proxy).

⚠️ That F1 figure needs care: it is near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. The conclusion may be right while the diagnostic is wrong — see [[vaep-conceding-classifier]].

### Axis 2: Target rarity

See [[rare-event-proxy-targets]]. Five frameworks substitute a denser proxy for goals. **The proxy is not a neutral substitution; it reorganises the model around itself** — [[c-obso|C-OBSO]] weights the goalkeeper *double*, [[xsot|Yeung & Fujii]] remove him entirely, because a save still counts as *on target*.

### Axis 3: Whose value — actor, receiver, or beneficiary

Three distinct answers where the vault long had one. [[c-obso]] correlates 0.45 with salary on players where [[obso|OBSO]] (−0.28) and goals (−0.23) do not. See [[space-creation]].

### Axis 4: Observed policy or optimal policy

Every framework except [[xsot|Yeung & Fujii]] estimates value under the **observed** policy. The stated reason is that the optimal-policy counterfactual is unfounded — no data contains perfect play.

[[game-theory|Yeung & Fujii]] show when that objection bites: restrict to a **two-action game** and payoffs for unobserved profiles become *estimable* rather than extrapolated. **The barrier is the size of the action space, not the observational nature of the data.** Coarsening is the price. See [[policy-modelling]].

### Axes 5–7: intent vs outcome, attributable possession, realised vs available

Whether the model sees how the action turned out ([[intent-vs-outcome-valuation]]); whether contested events are visible ([[symmetrical-duel-valuation]]); whether unrealised options can be valued ([[probability-surface|pass surfaces]]).

### Credit assignment over time

Six positions, from [[vaep]]'s fixed $k=10$ action window through [[expected-value-possession-framework|Fernández et al.'s]] hard 15 s cutoff to [[temporal-discounting|Shelopugin's]] geometric decay and [[obso|OBSO's]] single next-event horizon.

**Five frameworks carry an asserted free parameter and none reports a sensitivity analysis.** [[obso|Spearman]] is the exception, and the reason is structural: parameters with **physical units** admit priors from previous measurement. Note the five are not the same kind of parameter — horizons are likely self-limiting while $\gamma$ and $C$ are not. See [[free-parameters-load-bearing]] and [[model-selection]].

## Observed Versus Optimal: What the Two Results Do and Do Not Share

Two frameworks with nothing methodologically in common each report a large gap between observed behaviour and their model's optimum.

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value surface | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | Game-theoretic payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |

**Correction, 2026-07-27.** An earlier revision of this section described the two as "locating the divergence in the same place." That was too strong. They measure different things:

- **Fernández et al.** measure suboptimal **targeting *within* an action** — given a pass, it often does not go to the highest-value destination. The action type is not in question.
- **Yeung & Fujii** measure suboptimal **selection *between* actions** — given a shot situation, the shooter shoots when the equilibrium says pass. The destination is not in question.

So **"shooters shoot too much" is Yeung & Fujii's claim alone.** What the two jointly support is weaker and still substantial: a large observed-versus-optimal gap appears under two unrelated methods, on different leagues, different data types and different decision granularities.

That is convergence on the **existence** of a gap, not on its cause — and the cause matters, because at least three explanations produce a large gap with no decision error present:

1. **Average-player models applied to specific players.** An elite finisher *should* shoot more than an average-player equilibrium prescribes, and would be scored as over-shooting for doing the right thing.
2. **Execution difficulty is unmodelled.** Both value the *intent* of the best option, not the difficulty of playing it. The gap may be the price of reliability.
3. **Equilibrium assumes a rational opponent.** If defenders systematically fail to block, the shooter's true best response shifts toward shooting.

Neither paper tests whether the gap predicts anything outside its own model. See [[observed-versus-optimal-decisions]].

## Off-Ball Valuation: Four Mechanisms

A player has the ball for roughly **3 of 90 minutes**.

| | Surface at position | 22 positions in state | Predicted reference | Physical surface |
|---|---|---|---|---|
| Values | The receiver | The defence | The creator | The receiver |
| Output unit | Player | **Team only** | Player | Player |
| Example | EPV surface | [[vdep]] | [[c-obso]] | [[obso]] |

**The individuating ingredient is the counterfactual, not the data.** VDEP and C-OBSO use comparable tracking data; VDEP produces one number per configuration, C-OBSO intervenes on one *named* player. See [[counterfactual-baseline]].

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Predicts | Next event | Event + time | Event + exact $(x,y)$ | Event + lineup | **All agents' positions** |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**Forecasting produces metrics as a by-product**, needing no outcome labels, so goal sparsity never bites. See [[event-prediction]].

### The handcrafted-features question

Seq2Event degrades *without* engineered geometry; [[sig-model]] degrades *with* it; [[xsot|Yeung & Fujii]] find a [[theory-based-modelling|theory-based feature]] essential and raw coordinates actively harmful.

A rule reconciling all three is proposed on [[representation-learning]]: **encode structure the representation cannot recover *and* the data cannot support learning; encode nothing else.** Note this is a reconciliation *this vault constructed*, not a finding any source states, and it is untested — see [[handcrafted-features-rule]].

The same sample-size logic explains why tree ensembles **won** on VAEP's 8.5M actions and came **last** on Yeung & Fujii's 2,575 shots. See [[gradient-boosting]].

## Tasks 3–5: Clustering, Counterfactual, Tactical

[[football-event-sequences-point-process-mixture|Possession clustering]] into tactical types, validated by [[adjusted-rand-index|ARI]] and BIC. [[scoutgpt|Generative simulation]] against [[transfer-performance-prediction|regression on context]] — regression to narrow a market, simulation to discriminate among survivors. And [[tactical-analysis|pressing analysis]], which has the **weakest validation in the vault** and unusually persuasive output.

## Metrics Beat Outcomes at Predicting Outcomes

Established at both levels by independent sources seven years apart.

**Player level** ([[beyond-expected-goals|Spearman, 2018]]):

| Predictor | Next-match goals |
|---|---|
| **[[obso\|OBSO]]** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**Team level** ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]): LPV 0.28 and HPUS 0.26 against next-match goals, versus xG 0.17 and goals 0.11.

**Goals are the worst predictor of future goals in both.** A third instance: [[xsot|xSOT]] correlates 0.58 with average goals where [[expected-goals|xG]] manages 0.46.

Spearman's remains the strongest — **player-level**, against an **independent outcome**, and **beating the outcome's own lagged value**. Contrast [[epv-control-duel-skills-football|Shelopugin's]] next-season forecast, which predicts the metric's own future value; self-prediction shows persistence, not validity.

Four validation checks in ascending strength: self-prediction → cross-horizon consistency ([[vdep]]) → external outcome (Spearman) → external criterion outside the pipeline ([[c-obso]] vs salary).

A caution on the reliability side of this: [[split-half-reliability]] and [[performance-volatility]] turn out to be measuring the **same variance component** with opposite interpretations, so they are not independent evidence. See [[within-season-variation-noise-or-signal]].

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by four off-ball mechanisms. Still uncovered: **errors of omission**, movement beyond short windows.
3. **Individual defensive credit** — addressed in the literature (Umemoto & Fujii, 2023), not held here.
4. **No ground truth**, which is why reliability, predictive validity, calibration, imbalance-robust metrics and external criteria have all become substitute tests.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models** — [[c-obso]] predicts 3 of 22 players.
7. **Component-level divergence, invisible in framework-level comparison.** Two uncompared [[pitch-control-traditions-compared|pitch-control traditions]], four unbenchmarked [[shot-value-formulations-compared|shot-value formulations]], and [[tracking-error-propagation|tracking error]] that nobody propagates. All three are shared ingredients that differ silently between frameworks whose outputs *are* compared — the benchmarking gap one level down, and worse there.
8. **Strategy-space coarsening** is the price of prescription.
9. **Price is absent everywhere.**
10. **No cross-framework benchmarking.** Yeung & Fujii compare against no decision-making baseline either.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist; [[scoutgpt|simulation]] for fit; [[player-development-curve|PDC]] for trajectory.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two metrics here with external validation.
- **Coaching a decision, not describing one** → [[xsot|the SPC framework]], the only prescriptive tool here — with the caveat above about what its gap does and does not establish.
- **Assessing a defence** → [[vdep]] (team level only).
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]]; [[obso|OBSO]] for ranking moments at far lower cost.
- **Valuing what was available** → [[probability-surface|pass surfaces]].
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Aerial or physical targets** → [[duel-skill-rating]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]] over raw inputs, and avoid tree ensembles.
- **Shot quality alone** → xG, a *component* of the others rather than a competitor.

## Open Questions

Each has a page carrying the analysis:

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[game-theory]] · [[xsot]] · [[theory-based-modelling]] · [[policy-modelling]] · [[reinforcement-learning]]
- [[obso]] · [[c-obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[representation-learning]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]] · [[player-rating-time-series]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]]
- [[william-spearman]] · [[keisuke-fujii]] · [[calvin-yeung]] · [[javier-fernandez]]
