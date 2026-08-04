---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

> **On provenance.** A synthesis generates claims that no single source states. The load-bearing ones are marked `^[generated]` at their point of use, with the fuller argument — and what would falsify it — on the linked concept page. Claims marked `absence:` have a built-in expiry date. Eight open questions have their own pages, listed at the foot.

## The Six Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision or position | [[physics-based-pass-probabilities\|Spearman et al. (2017)]], [[xsot\|Yeung & Fujii]], [[drso\|DRSO]] |

**Prescription is genuinely a sixth task**^[generated: the six-task division is this vault's organising scheme; no source proposes it. rests-on: absence:no-source-proposes-a-task-taxonomy] — **and it is the oldest, not the newest.**

> **Correction, 2026-07-27.** This section previously presented prescription as a recent development with two instances. It has three, and the earliest is the **oldest football source in the vault**. [[physics-based-pass-probabilities|Spearman et al. (2017)]] section 6.2 uses simulated annealing to find the ball velocity maximising reception probability for an intended receiver, then perturbs it to give $\mu_P + \sigma_P$ ("if well-kicked") and $\mu_P - \sigma_P$ ("if poorly-kicked"). High-high means easy; high-low means feasible but requiring skill.

| Instance | Year | Optimises over | Choice set |
|---|---|---|---|
| **[[physics-based-pass-probabilities\|Hypothetical passing]]** | **2017** | Ball velocity vector | Continuous, searched by annealing |
| [[xsot\|SPC framework]] | 2024 | Action | Two: shoot or pass |
| [[drso\|DRSO]] | 2023 | Defender position | Four grid vertices |

The 2017 instance is also the only one that **did not scale** — the authors report computational cost prevented large-scale application, so no metric was built from it. Seven years later, both successors solved the same problem by **coarsening the choice set to a handful of options** rather than by cheaper search.

That is the price of prescription, visible across all three: **you can only prescribe over a choice set you can enumerate.**

## Task 1: Valuation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[gvdep]] | [[obso]] | [[c-obso]] | [[drso]] | [[xsot\|xSOT]] |
|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | **Defence** | **Defence** | Attack | Attack | **Defence** | **Both** |
| **Whose value** | Actor | Actor | Actor | Team | Team | Receiver | **Teammate's** | **The defender** | Actor |
| **Mechanism** | DP | Boosting | Neural ×9 | XGBoost ×2 | XGBoost ×4 | **Physical** | **Counterfactual** | **Counterfactual, no ML** | **Hybrid + game** |
| **Reference** | — | — | — | — | — | — | Predicted | **Optimal** | Optimal |
| **[[interpretability]]** | **High** | Low | Moderate | Moderate | Moderate | **High** | Moderate | **High** | Moderate |
| **Reported unit** | Player | Player | Situation | **Team** | **Team** | Player | Player | **Team** | Situation |
| **Cost** | Trivial | Modest | Modest | Modest | **Low** | **Low** | High | **Low** | Low |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]] and [[drso|DRSO]] are the exceptions: tracking-based, interpretable and cheap, because both are **physical rather than learned**.

⚠️ Reliability evidence is thinner than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric here has a reported split-half reliability.^[generated: rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ re-checked on the GVDEP, DRSO and Spearman-2017 ingests: still holds.] The metrics best suited to finding undervalued players are the ones whose stability is least known.

### Axis 1: Perspective

VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15 on comparable data. [[physics-based-pass-probabilities|Spearman et al.]] demonstrate the mechanism directly — their accuracy rises 80.5% → 81.9% purely by moving a cutoff from 0.5 to 0.27. See [[vaep-conceding-classifier]].

> ### `offensive-bias-four-causes`
> **Four distinct causes with four different remedies:** definitional, data, modelling choice, statistical.
> ^[generated: no source enumerates these. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question. Declared on [[action-valuation]].]

### Axes 2–4

**Target rarity** — the proxy reorganises the model around itself; [[c-obso|C-OBSO]] weights the goalkeeper double, [[xsot|Yeung & Fujii]] remove him entirely. **Whose value** — four answers where the vault long had one. **Observed or optimal policy** — see the prescription section above.

### Free parameters

**Four frameworks still carry an asserted parameter with no sensitivity analysis** — $\gamma$, $\epsilon$, $k$, 4 s.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ narrowed at GVDEP, re-checked at DRSO and Spearman 2017, both of which fit or sweep *other* parameters. Declared on [[model-selection]].]

**VDEP's $C$ was superseded** by [[gvdep|GVDEP]]. [[obso|Spearman]] remains the exception, because `physical-units-admit-priors`^[generated: declared on [[model-selection]]] — and the 2017 paper strengthens that claim materially, since its fitted values with stated stat and syst errors are demonstrably what the 2018 priors inherit.

## Observed Versus Optimal

| Source | Method | Gap |
|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value surface | 0.032 → 0.112 |
| [[xsot\|Yeung & Fujii]] | Game-theoretic payoffs | 0.0866 → 0.2456 |
| [[drso\|DRSO]] | Counterfactual position | −0.040 to −0.052 per team |

The first two measure different things — **targeting within** an action versus **selection between** actions — so "shooters shoot too much" is Yeung & Fujii's claim alone. All three converge on the **existence** of a gap, not its cause. See [[observed-versus-optimal-decisions]].

## Off-Ball Valuation: Four Mechanisms

| | Surface at position | Positions in state | Predicted reference | Optimal position |
|---|---|---|---|---|
| Values | The receiver | The defence | The creator | **The defender** |
| Reported unit | Player | **Team only** | Player | **Team** |
| Example | EPV surface, [[obso]] | [[vdep]], [[gvdep]] | [[c-obso]] | **[[drso]]** |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO; necessity unproven. rests-on: claim:counterfactual-individuates]

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning.^[generated: declared on [[representation-learning]]; never tested against a case it was not built to fit. rests-on: claim:handcrafted-features-rule] A working heuristic, not a finding.

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]): OBSO 0.26 against next-match goals, shots 0.17, goals 0.12. **Team level**: LPV 0.28, HPUS 0.26, xG 0.17, goals 0.11. **Goals are the worst predictor of future goals in both.**

Four validation checks in ascending strength: self-prediction → cross-horizon consistency → external outcome → external criterion outside the pipeline.

A fifth, rarely available and stronger than all of them: **validation of a component against a directly observable quantity.** [[physics-based-pass-probabilities|Spearman et al.]] test PPCF against who actually received 5,471 held-out passes (81% / 68%). Almost nothing else here has an observable ground truth to check against — which is why the substitute tests above exist at all.

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by four off-ball mechanisms. Still uncovered: **errors of omission**.
3. **Individual defensive credit is computed but not reported.** [[drso|DRSO]] computes $Diff_{opt-obs}$ per named defender; every published result averages to teams. One aggregation step, not a missing method. See [[defensive-valuation]].
4. **No ground truth** for most quantities — hence the substitute tests, and hence how much weight the one directly validated component carries.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models**; GVDEP suggests three or four players may suffice.
7. **Component-level divergence, invisible in framework-level comparison.** Two [[pitch-control-traditions-compared|pitch-control traditions]], four [[shot-value-formulations-compared|shot-value formulations]], [[tracking-error-propagation|tracking error]] nobody propagates, and PPCF parameters two papers attribute to the wrong Spearman paper (the values are right; see [[obso]]).

   **A validation asymmetry underlies the first of these:** PPCF is fitted and tested against actual pass receivers; the Gaussian influence model has parameters set to 1 and is validated against nothing directly. The two are not equally warranted, so their disagreement is not symmetrically informative.

8. **Strategy-space coarsening** is the price of prescription, in all three instances.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent**, below.

> ### `no-cross-framework-benchmarking`
> **Comparison happens within research lines and never across them.**
> ^[generated: rests-on: absence:no-held-source-benchmarks-across-frameworks, source:gvdep-vs-vdep-comparison — ⚠️ weakened at GVDEP; re-checked at DRSO and Spearman 2017, neither of which compares against anything. Highest-value claim to re-check on every ingest.]

[[gvdep|GVDEP]] compares against [[vdep|VDEP]] on identical data — but it is **same-group, own-predecessor** work within one lineage. Groups benchmark against themselves; nobody benchmarks against a competitor. This is why the tables above compare **design characteristics rather than results**.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression]] to shortlist; [[scoutgpt|simulation]] for fit.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two with external validation.
- **Coaching a decision** → [[xsot|SPC]]. **Coaching a position** → [[drso|DRSO]]. **Coaching a pass** → [[physics-based-pass-probabilities|hypothetical passing]], if you can afford the search.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]]. Team level.
- **Working from broadcast video rather than a tracking licence** → [[gvdep|GVDEP]], [[drso|DRSO]], [[obso|OBSO]].
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]]; avoid tree ensembles.
- **Any thresholded classifier** → tune the cutoff. 0.5 is a convention; see [[class-imbalance-evaluation]].

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
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]] · [[representation-learning]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]] · [[recruitment]]
- [[william-spearman]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[javier-fernandez]]
