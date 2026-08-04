---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 45%
  inferred: 35%
  generated: 15%
  imported: 2%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **six distinct tasks**, each answering a different question, with different data requirements and validation strategies.

> **On provenance.** A synthesis generates claims that no single source states. The load-bearing ones are marked `^[generated]` at their point of use, with the fuller argument — and what would falsify it — on the linked concept page. Claims marked `absence:` have a built-in expiry date: they hold only until a source is acquired that contradicts them, which is how most of this vault's corrections have arrived. Eight open questions have their own pages, listed at the foot.

## The Six Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[drso]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision or position | [[xsot\|Yeung & Fujii]], **[[drso\|DRSO]]** |

**Prescription is genuinely a sixth task.**^[generated: the six-task division is this vault's organising scheme; no source proposes it. rests-on: absence:no-source-proposes-a-task-taxonomy]

It now has **two instances arriving independently**: [[xsot|Yeung & Fujii]] solve for the optimal *action* via [[game-theory|Nash equilibrium]]; [[drso|DRSO]] solves for the optimal *position* via counterfactual search. Both output advice rather than a score, and both come from the same research group by different routes.

## Task 1: Valuation

All frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[gvdep]] | [[obso]] | [[c-obso]] | [[drso]] | [[xsot\|xSOT]] |
|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | **Defence** | **Defence** | Attack | Attack | **Defence** | **Both** |
| **Whose value** | Actor | Actor | Actor | Team | Team | Receiver | **Teammate's** | **The defender** | Actor |
| **Target** | Goal | Goal | Next goal | **Recovery** | **Recovery** | Goal | Goal | Goal | **Shot on target** |
| **Mechanism** | DP | Boosting | Neural ×9 | XGBoost ×2 | XGBoost ×4 | **Physical** | **Counterfactual** | **Counterfactual, no ML** | **Hybrid + game** |
| **Reference** | — | — | — | — | — | — | Predicted | **Optimal** | Optimal |
| **[[interpretability]]** | **High** | Low | Moderate | Moderate | Moderate | **High** | Moderate | **High** | Moderate |
| **Reported unit** | Player | Player | Situation | **Team** | **Team** | Player | Player | **Team** | Situation |
| **Cost** | Trivial | Modest | Modest | Modest | **Low** | **Low** | High | **Low** | Low |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]] and [[drso|DRSO]] are the exceptions: both tracking-based, both highly interpretable, both cheap, because both are **physical rather than learned**.

⚠️ The reliability evidence is thinner than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric in this vault has a reported split-half reliability.^[generated: established by checking every such framework held here. rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ re-checked on the GVDEP and DRSO ingests, 2026-07-27: still holds. Also on [[off-ball-value]] and [[defensive-valuation]].] Since reliability is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

### Axis 1: Perspective — attacking or defending

[[football-defence-evaluation-vdep|Toda et al.]] report VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15 on comparable data, confirming the zero is not a fixed property of the model. See [[vaep-conceding-classifier]].

> ### `offensive-bias-four-causes`
>
> **Offensive bias has four distinct causes with four different remedies:** definitional, data, modelling choice, and statistical.
>
> ^[generated: no source enumerates these. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question, so the fourth cause is the least secure. Declared on [[action-valuation]].]

### Axis 2: Target rarity

See [[rare-event-proxy-targets]]. **The proxy is not a neutral substitution; it reorganises the model around itself** — [[c-obso|C-OBSO]] weights the goalkeeper *double*, [[xsot|Yeung & Fujii]] remove him entirely, because a save still counts as *on target*.

### Axis 3: Whose value — actor, receiver, beneficiary, or defender

Four distinct answers where the vault long had one. [[c-obso]] correlates 0.45 with salary on players where [[obso|OBSO]] (−0.28) and goals (−0.23) do not. See [[space-creation]].

### Axis 4: Observed policy or optimal policy

Most frameworks estimate value under the **observed** policy, on the ground that the optimal-policy counterfactual is unfounded.

Two escape it, by shrinking the choice set until every option is enumerable: [[game-theory|Yeung & Fujii]] to **two actions**, [[drso|DRSO]] to **four grid vertices**. **The barrier is the size of the action space, not the observational nature of the data** — and coarsening is the price in both cases. See [[policy-modelling]].

### Credit assignment and free parameters

**Four frameworks still carry an asserted free parameter with no sensitivity analysis** — $\gamma$, $\epsilon$, $k$, and C-OBSO's 4 s.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ narrowed at the GVDEP ingest and re-checked at DRSO, which verifies five *velocity* settings but no horizon parameter. Declared on [[model-selection]].]

**VDEP's $C$ has been superseded** by [[gvdep|GVDEP]]'s VAEP-derived weights — the vault's only instance of an asserted parameter fixed by principled derivation. [[obso|Spearman]] remains the exception on the others, because `physical-units-admit-priors`^[generated: declared on [[model-selection]]; rests on a single source's parameter table]. See [[free-parameters-load-bearing]].

## Observed Versus Optimal

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value surface | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | Game-theoretic payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |
| [[drso\|DRSO]] | Counterfactual position | — | — | −0.040 to −0.052 per team |

The first two measure different things — Fernández et al. suboptimal **targeting within** an action, Yeung & Fujii suboptimal **selection between** actions — so **"shooters shoot too much" is Yeung & Fujii's claim alone.**

DRSO adds a third granularity: suboptimal **positioning**, where no action is chosen at all. What all three share is convergence on the **existence** of a gap, not its cause. At least three explanations produce one with no decision error present. See [[observed-versus-optimal-decisions]].

## Off-Ball Valuation: Four Mechanisms

A player has the ball for roughly **3 of 90 minutes**.

| | Surface at position | Positions in state | Predicted reference | Optimal position |
|---|---|---|---|---|
| Values | The receiver | The defence | The creator | **The defender** |
| Reported unit | Player | **Team only** | Player | **Team** |
| Example | EPV surface, [[obso]] | [[vdep]], [[gvdep]] | [[c-obso]] | **[[drso]]** |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. **Supported by DRSO**: same group, comparable data, and intervening on one named defender produces his own number where VDEP and GVDEP stay at team level. Necessity unproven — Shapley-style attribution would individuate without intervening. rests-on: claim:counterfactual-individuates]

[[gvdep|GVDEP]] bounds how much data the second column needs: **ball-gain prediction saturates at three or four players**, and the other three targets gain nothing from player positions at all.

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Predicts | Next event | Event + time | Event + exact $(x,y)$ | Event + lineup | **All agents' positions** |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**Forecasting produces metrics as a by-product**, needing no outcome labels. See [[event-prediction]].

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning; encode nothing else.^[generated: declared on [[representation-learning]]. A reconciliation constructed here, never tested against a case it was not built to fit. rests-on: claim:handcrafted-features-rule] Treat as a working heuristic.

GVDEP supplies a fourth consistent data point: its concedes classifier gets **worse** as player positions are added (F1 0.15 → 0.08), with 186 positives. Same logic as tree ensembles winning on 8.5M actions and coming last on 2,575 shots. See [[gradient-boosting]].

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]): OBSO 0.26 against next-match goals, shots 0.17, goals 0.12.

**Team level** ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]): LPV 0.28, HPUS 0.26, xG 0.17, goals 0.11.

**Goals are the worst predictor of future goals in both.** A third instance: [[xsot|xSOT]] correlates 0.58 with average goals where [[expected-goals|xG]] manages 0.46.

Four validation checks in ascending strength: self-prediction → cross-horizon consistency ([[vdep]]) → external outcome (Spearman) → external criterion outside the pipeline ([[c-obso]] vs salary).

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by four off-ball mechanisms. Still uncovered: **errors of omission**, movement beyond short windows.
3. **Individual defensive credit is computed but not reported.**

   > **Superseded, 2026-07-27.** This limitation read "addressed in the literature, not held here" through six log entries. [[team-defense-positioning-counterfactuals|Umemoto & Fujii (2023)]] is now held, and the claim splits:
   >
   > - No held framework **computes** per-defender credit — **false**. [[drso|DRSO]] computes $Diff_{opt-obs}$ per named defender.
   > - No held framework **reports** per-defender credit — **still true**. Every result averages three defenders, then events, then teams.
   >
   > The gap is one aggregation step, not a missing method. See [[defensive-valuation]].

4. **No ground truth**, which is why reliability, predictive validity, calibration, imbalance-robust metrics and external criteria have all become substitute tests — and why two of them turn out not to be independent.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models** — [[c-obso]] predicts 3 of 22 players; GVDEP suggests three or four may suffice, so this may be less binding than assumed.
7. **Component-level divergence, invisible in framework-level comparison.** Two uncompared [[pitch-control-traditions-compared|pitch-control traditions]], four unbenchmarked [[shot-value-formulations-compared|shot-value formulations]], and [[tracking-error-propagation|tracking error]] nobody propagates. **A fourth instance:** two Fujii-group papers use PPCF parameters that match neither Spearman's fitted values nor his stated priors, while citing him — see [[obso]].
8. **Strategy-space coarsening** is the price of prescription, for both instances.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent**, below.

> ### `no-cross-framework-benchmarking`
>
> **Comparison happens within research lines and never across them.**
>
> ^[generated: the vault's most-repeated absence claim, and the basis for its comparison tables resting on design characteristics rather than results. rests-on: absence:no-held-source-benchmarks-across-frameworks, source:gvdep-vs-vdep-comparison — ⚠️ weakened at the GVDEP ingest; re-checked at DRSO, which compares against nothing at all. The single highest-value claim to re-check on every ingest.]

[[gvdep|GVDEP]] compares directly against [[vdep|VDEP]] on identical data — a genuine like-for-like comparison, and the first here. But it is **same-group, own-predecessor** work, and within one lineage: nothing compares VDEP, GVDEP or DRSO against [[vaep]]'s defensive half, [[c-obso]], or any framework outside the Fujii group.

Groups benchmark against themselves; nobody benchmarks against a competitor. This is why the tables above compare **design characteristics rather than results**.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression]] to shortlist; [[scoutgpt|simulation]] for fit; [[player-development-curve|PDC]] for trajectory.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two metrics with external validation.
- **Coaching a decision** → [[xsot|the SPC framework]]. **Coaching a position** → [[drso|DRSO]]. The only two that output advice rather than a score.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]] — principled weighting, no full-tracking requirement. Team level.
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]]; [[obso|OBSO]] at far lower cost.
- **Working from broadcast video rather than a tracking licence** → [[gvdep|GVDEP]], [[drso|DRSO]] and [[obso|OBSO]], all designed to minimise data requirements.
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]] over raw inputs; avoid tree ensembles.
- **Shot quality alone** → xG, a *component* of the others.

## Open Questions

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[game-theory]] · [[xsot]] · [[drso]] · [[theory-based-modelling]] · [[policy-modelling]] · [[reinforcement-learning]]
- [[obso]] · [[c-obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[representation-learning]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]] · [[recruitment]]
- [[william-spearman]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[javier-fernandez]]
