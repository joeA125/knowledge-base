---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[space-occupation-gain\|SOG]], [[martingale-epv]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision or position | [[physics-based-pass-probabilities\|Spearman et al. (2017)]], [[xsot\|Yeung & Fujii]], [[drso\|DRSO]] |

**Prescription is genuinely a sixth task**^[generated: the six-task division is this vault's organising scheme; no source proposes it. rests-on: absence:no-source-proposes-a-task-taxonomy] — **and it is the oldest, not the newest.**

| Instance | Year | Optimises over | Choice set |
|---|---|---|---|
| **[[physics-based-pass-probabilities\|Hypothetical passing]]** | **2017** | Ball velocity vector | Continuous, searched by annealing |
| [[drso\|DRSO]] | 2023 | Defender position | Four grid vertices |
| [[xsot\|SPC framework]] | 2024 | Action | Two: shoot or pass |

The 2017 instance is the only one that **did not scale** — computational cost prevented large-scale application, so no metric was built. Both successors solved the same problem by **coarsening the choice set** rather than by cheaper search. **You can only prescribe over a choice set you can enumerate.**

## Task 1: Valuation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[gvdep]] | [[obso]] | [[space-occupation-gain\|SOG]] | [[c-obso]] | [[drso]] | [[xsot\|xSOT]] |
|---|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | **Def** | **Def** | Attack | Attack | Attack | **Def** | **Both** |
| **Whose value** | Actor | Actor | Actor | Team | Team | Receiver | **Occupier** | **Teammate's** | **Defender** | Actor |
| **Measures** | Level | Level | Level | Level | Level | Level | **Rate** | Rate | Level | Level |
| **Mechanism** | DP | Boosting | Neural ×9 | XGB ×2 | XGB ×4 | **Physical** | **Control × value** | **Counterfactual** | **Counterfactual** | **Hybrid + game** |
| **[[interpretability]]** | **High** | Low | Moderate | Moderate | Moderate | **High** | **High** | Moderate | **High** | Moderate |
| **Reported unit** | Player | Player | Situation | **Team** | **Team** | Player | Player | Player | **Team** | Situation |
| **Cost** | Trivial | Modest | Modest | Modest | **Low** | **Low** | Modest | High | **Low** | Low |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]], [[space-occupation-gain|SOG]] and [[drso|DRSO]] are the exceptions: tracking-based, interpretable and cheap, because all three are **physical or geometric rather than learned**.

⚠️ Reliability evidence is thinner than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric here has a reported split-half reliability.^[generated: rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ re-checked on the GVDEP, DRSO, Spearman-2017 and Wide Open Spaces ingests: still holds.] The metrics best suited to finding undervalued players are the ones whose stability is least known.

### Axis 1: Perspective

VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15, and [[physics-based-pass-probabilities|Spearman et al.]] demonstrate the mechanism directly — accuracy rising 80.5% → 81.9% purely by moving a cutoff from 0.5 to 0.27. See [[vaep-conceding-classifier]].

> ### `offensive-bias-four-causes`
> **Four distinct causes with four different remedies:** definitional, data, modelling choice, statistical.
> ^[generated: no source enumerates these. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question. Declared on [[action-valuation]].]

### Axes 2–4

**Target rarity** — the proxy reorganises the model around itself. **Whose value** — five answers now: actor, receiver, occupier, beneficiary's creator, defender. **Observed or optimal policy** — see the prescription table above.

### Free parameters

**Eight frameworks carry an asserted parameter with no sensitivity analysis**, up from four across three ingests.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ re-checked and strengthened at the Wide Open Spaces ingest, which adds four. Declared on [[model-selection]].]

They are of **four kinds**, and the newest are the worst-behaved. Horizons ($\epsilon$, $k$, 4 s, $w$) are likely self-limiting; $\gamma$ is a shape parameter spanning two orders of magnitude; and SOG/SGG's $\delta$ and $\alpha$ are **gates** — a gate that is slightly wrong produces the *wrong set of events*, not slightly wrong values. **VDEP's $C$ has been superseded** by [[gvdep|GVDEP]], the vault's only asserted parameter fixed by principled derivation. See [[free-parameters-load-bearing]].

## Off-Ball Valuation: Five Mechanisms

A player has the ball for roughly **3 of 90 minutes** — quoted by both pitch-control origin papers.

| | Surface at position | Control × value | Positions in state | Predicted reference | Optimal position |
|---|---|---|---|---|---|
| Values | The **receiver** | The **occupier** | The **defence** | The **creator** | The **defender** |
| Measures | A level | **A rate** | A level | A rate | A level |
| Reported unit | Player | Player | **Team only** | Player | **Team** |
| Example | EPV surface, [[obso]] | [[space-occupation-gain\|SOG]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO. **Nearest counter-example:** [[space-occupation-gain|SGG]] produces per-player creation credit by *spatial predicate*, without a counterfactual — though it attributes by co-occurrence rather than establishing what would otherwise have happened. rests-on: claim:counterfactual-individuates]

**Space creation is now covered twice, by unrelated mechanisms.** [[space-occupation-gain|SGG]] (2018) attributes by **co-occurrence**; [[c-obso]] (2022/23) by **deviation from a predicted reference**. Neither cites the other, and they are directly comparable on a single match — an unrun test. Both independently find that **occupation and generation are distinct skills**, in different leagues. See [[space-creation]].

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning.^[generated: declared on [[representation-learning]]; never tested against a case it was not built to fit. rests-on: claim:handcrafted-features-rule] A working heuristic, not a finding.

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]): OBSO 0.26 against next-match goals, shots 0.17, goals 0.12. **Team level**: LPV 0.28, HPUS 0.26, xG 0.17, goals 0.11. **Goals are the worst predictor of future goals in both.**

Five validation modes, ascending: self-prediction → cross-horizon consistency → external outcome → external criterion outside the pipeline ([[c-obso]] vs salary) → **validation of a component against a directly observable quantity** ([[pass-probability-model|PPCF]] against 5,471 held-out pass receivers).

The fifth is rarely available, and its absence explains the other four: almost nothing here has an observable ground truth. [[wide-open-spaces-space-creation|Fernández & Bornn]] state it plainly — *"there is no existance of ground truth data regarding the quantification of spaces in soccer"* — and validate by expert video review instead.

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by five off-ball mechanisms. Still uncovered: **errors of omission**.
3. **Individual defensive credit is computed but not reported.** [[drso|DRSO]] computes it per named defender; every published result averages to teams. One aggregation step, not a missing method.
4. **No ground truth** for most quantities — hence the substitute tests, and hence how much weight the one directly validated component carries.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models.** [[c-obso]] predicts 3 of 22; [[space-occupation-gain|SOG/SGG]] analyse one match.
7. **Component-level divergence, invisible in framework-level comparison.** Two [[pitch-control-traditions-compared|pitch-control traditions]], four [[shot-value-formulations-compared|shot-value formulations]], [[tracking-error-propagation|tracking error]] nobody propagates, two uncompared space-creation methods, and PPCF parameters two papers attribute to the wrong Spearman paper.

   The pitch-control case is now **structurally explained**: neither tradition cites the other, and both position against [[voronoi-tessellation]]. They are **siblings framed against a common ancestor, not rivals** — which is why the comparison is nobody's responsibility and has stayed undone for eight years.

8. **Strategy-space coarsening** is the price of prescription, in all three instances.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent**, below.

> ### `no-cross-framework-benchmarking`
> **Comparison happens within research lines and never across them.**
> ^[generated: rests-on: absence:no-held-source-benchmarks-across-frameworks, source:gvdep-vs-vdep-comparison — ⚠️ weakened at GVDEP; re-checked at DRSO, Spearman 2017 and Wide Open Spaces, none of which compares against anything outside its own line.]

[[gvdep|GVDEP]] compares against [[vdep|VDEP]] on identical data — but it is **same-group, own-predecessor** work within one lineage. Groups benchmark against themselves; nobody benchmarks against a competitor. This is why the tables above compare **design characteristics rather than results**.

Note the contrast with the vault's computer-vision material, where [[camera-calibration-benchmarking|ProCC]] exists precisely to make cross-method comparison possible and consolidates a leaderboard across groups. **The benchmarking gap is a property of one research community, not of sports analytics as a field.**

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression]] to shortlist; [[scoutgpt|simulation]] for fit.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two with external validation.
- **Valuing movement rather than position** → [[space-occupation-gain|SOG]], the only metric here measuring a rate.
- **Coaching a decision** → [[xsot|SPC]]. **A position** → [[drso|DRSO]]. **A pass** → [[physics-based-pass-probabilities|hypothetical passing]], if you can afford the search.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]]. Team level.
- **Working from broadcast video rather than a tracking licence** → [[gvdep|GVDEP]], [[drso|DRSO]], [[obso|OBSO]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]]; avoid tree ensembles.
- **Any thresholded classifier** → tune the cutoff. See [[class-imbalance-evaluation]].

## Open Questions

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[game-theory]] · [[xsot]] · [[drso]] · [[theory-based-modelling]] · [[policy-modelling]] · [[reinforcement-learning]]
- [[obso]] · [[c-obso]] · [[space-occupation-gain]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]] · [[voronoi-tessellation]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]] · [[pass-probability-model]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]] · [[representation-learning]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]] · [[recruitment]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[calvin-yeung]]
