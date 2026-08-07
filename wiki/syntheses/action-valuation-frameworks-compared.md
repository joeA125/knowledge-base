---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, construct-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling, reinforcement-learning, multi-agent, action-space]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.9
provenance:
  extracted: 44%
  inferred: 35%
  generated: 16%
  imported: 2%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-08-07
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **six distinct tasks**, each answering a different question, with different data requirements and validation strategies.

> **On provenance.** A synthesis generates claims that no single source states. The load-bearing ones are marked `^[generated]` at their point of use, with the fuller argument — and what would falsify it — on the linked concept page. Claims marked `absence:` have a built-in expiry date. Eight open questions have their own pages, listed at the foot.

## The Six Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[space-occupation-gain\|SOG]], [[martingale-epv]], [[expected-value-possession-framework\|Fernández et al.]], [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |
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
| *([[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]], 2023 — **could**, and does not)* | 2023 | Action | **14, per player, per timestep** |

The 2017 instance is the only one that **did not scale** — computational cost prevented large-scale application, so no metric was built. Both successors solved the same problem by **coarsening the choice set** rather than by cheaper search. **You can only prescribe over a choice set you can enumerate.**

> **Added 2026-08-07.** Nakahara et al. sit awkwardly in that story. They *do* enumerate — 14 actions for all ten attackers at every timestep — and so hold the largest tractable prescriptive choice set in the vault. The observed-versus-maximum gap is one subtraction from their output tensor. **They report correlations with season goals instead.** So the constraint on prescription is not only enumerability; it is also that a group has to want to make the prescriptive claim. See [[observed-versus-optimal-decisions]].

## Task 1: Valuation

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[gvdep]] | [[obso]] | [[space-occupation-gain\|SOG]] | [[c-obso]] | [[drso]] | [[xsot\|xSOT]] | **[[action-valuation-multi-agent-reinforcement-learning\|Nakahara $Q$]]** |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | **Def** | **Def** | Attack | Attack | Attack | **Def** | **Both** | Attack (+conceding in reward) |
| **Whose value** | Actor | Actor | Actor | Team | Team | Receiver | **Occupier** | **Teammate's** | **Defender** | Actor | **All ten at once** |
| **Measures** | Level | Level | Level | Level | Level | Level | **Rate** | Rate | Level | Level | **Rate** |
| **Mechanism** | DP | Boosting | Neural ×9 | XGB ×2 | XGB ×4 | **Physical** | **Control × value** | **Counterfactual** | **Counterfactual** | **Hybrid + game** | **[[temporal-difference-learning\|TD/RL]]** |
| **[[interpretability]]** | **High** | Low | Moderate | Moderate | Moderate | **High** | **High** | Moderate | **High** | Moderate | **Low** |
| **Reported unit** | Player | Player | Situation | **Team** | **Team** | Player | Player | Player | **Team** | Situation | Player |
| **Cost** | Trivial | Modest | Modest | Modest | **Low** | **Low** | Modest | High | **Low** | Low | High |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]], [[space-occupation-gain|SOG]] and [[drso|DRSO]] are the exceptions: tracking-based, interpretable and cheap, because all three are **physical or geometric rather than learned**. Nakahara et al. sit at the far opposite corner — richest state, richest output, lowest interpretability, no physical structure at all.

### A fifth estimation style

Van Roy et al.'s taxonomy has three styles; this vault added *counterfactual* as a fourth and, on the 2026-08-07 ingest, **reinforcement-learning-based** as a fifth: learn $Q(s,a)$ over an explicit [[action-space-design|action space]] by bootstrapping against reward, and read counterfactual values off the learned function.^[generated: the fifth style is added on [[action-valuation]]; distinct from *action-based* because value is learned against reward by [[temporal-difference-learning|bootstrapping]] rather than regressed against an outcome label, and distinct from *counterfactual* because no reference baseline is differenced. rests-on: source:vanroy-three-style-taxonomy, source:nakahara-sarsa-td]

⚠️ Reliability evidence is thinner than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric here has a reported split-half reliability.^[generated: rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ re-checked on the GVDEP, DRSO, Spearman-2017, Wide Open Spaces and **Nakahara** ingests: still holds. **Six mechanisms, zero reliability estimates.**] The metrics best suited to finding undervalued players are the ones whose stability is least known.

**This claim stopped being a tidiness complaint on 2026-08-07.** See the next section: two off-ball metrics have now been compared and disagree, and without reliability figures the disagreement cannot be attributed.

### Axis 1: Perspective

VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15, and [[physics-based-pass-probabilities|Spearman et al.]] demonstrate the mechanism directly — accuracy rising 80.5% → 81.9% purely by moving a cutoff from 0.5 to 0.27. See [[vaep-conceding-classifier]].

A third position exists: Nakahara et al. put conceding in the **reward** ($-1$ if the opponent scores immediately after the possession ends) rather than in a classifier. Because reward attaches to a possession rather than an event, the base rate is diluted by roughly the length of a possession. Untested — they report no defensive results — but it is the only structural response to the rarity problem here that is neither a proxy substitution nor a threshold adjustment.

> ### `offensive-bias-four-causes`
> **Four distinct causes with four different remedies:** definitional, data, modelling choice, statistical.
> ^[generated: no source enumerates these. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question. **Checked 2026-08-07:** Nakahara et al.'s ranking is topped by two zero-goal centre-backs, the reverse of the van Dijk result, which is consistent with cause 1 (definitional). A demonstration rather than a test — no held source computes VAEP and $Q$ on the same players. Declared on [[action-valuation]].]

### Axes 2–5

**Target rarity** — the proxy reorganises the model around itself. **Whose value** — six answers now: actor, receiver, occupier, beneficiary's creator, defender, and *every attacker simultaneously*. **Observed or optimal policy** — see the prescription table above. **[[action-space-design|Action space]]** — new as an axis, and arguably the most consequential, because it fixes which counterfactuals a framework can pose at all.

### Free parameters

**Fourteen frameworks-worth of asserted parameters carry no sensitivity analysis**, up from four across four ingests.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ re-checked at the Nakahara ingest, which adds six. Still holds: the one two-point ablation there is on a prior-strength parameter, not a horizon. Declared on [[model-selection]].]

They are of **five kinds**, and the newest is the most consequential. Horizons ($\epsilon$, $k$, 4 s, $w$, $T_{max}$) are likely self-limiting; $\gamma$ is a shape parameter spanning two orders of magnitude; SOG/SGG's $\delta$ and $\alpha$ and Nakahara's velocity thresholds are **gates** — a gate that is slightly wrong produces the *wrong set of events*, or the *wrong action label*, not slightly wrong values. **VDEP's $C$ has been superseded** by [[gvdep|GVDEP]], the vault's only asserted parameter fixed by principled derivation.

The fifth kind is **prior strength**: $\lambda_1$ in [[action-supervision]], which weights an imitation loss against a value loss.

> ### `optimality-gap-is-tunable`
> **In any framework that regularises a value function toward observed behaviour, the measured gap between observed and optimal action is partly a function of the regularisation weight, not of the players.**
> ^[generated: drawn from the source's own description of the $\lambda_1$ extremes; the general claim is not stated there. Declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff]

Every other parameter here, if wrong, produces wrong values or the wrong event set. A wrong $\lambda_1$ produces **a wrong conclusion about whether footballers decide well** — which is the closest thing this literature has to an actionable finding. See [[free-parameters-load-bearing]] and [[observed-versus-optimal-decisions]].

## Off-Ball Valuation: Six Mechanisms

A player has the ball for roughly **3 of 90 minutes** — quoted by both pitch-control origin papers, and by Nakahara et al. from the other side (~87 of 90 off the ball).

| | Surface at position | Control × value | Positions in state | Predicted reference | Optimal position | **Learned $Q$** |
|---|---|---|---|---|---|---|
| Values | The **receiver** | The **occupier** | The **defence** | The **creator** | The **defender** | **The mover** |
| Measures | A level | **A rate** | A level | A rate | A level | **A rate** |
| Needs a baseline | Yes (surface) | Yes (surface) | No | **Yes** | **Yes** | **No** |
| Reported unit | Player | Player | **Team only** | Player | **Team** | **Player** |
| Example | EPV surface, [[obso]] | [[space-occupation-gain\|SOG]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |

The sixth is the only route requiring **no reference to difference against**: a learned $Q$ is defined over actions nobody took, so counterfactual values come free. "Free" means "not separately estimated", not "well estimated" — on-policy SARSA never targets them, so they are shaped by [[action-supervision]] and network smoothness.

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. Supported by DRSO. **Weakened 2026-08-07:** route 6 individuates by *agent decomposition* — one value function per player — which is a second non-counterfactual individuator alongside SGG's spatial predicate. Two independent counter-examples is materially more pressure than one, though neither is decisive: SGG attributes by co-occurrence and per-agent splitting is a modelling choice rather than a measurement. rests-on: claim:counterfactual-individuates]

**Space creation is now covered twice, by unrelated mechanisms.** [[space-occupation-gain|SGG]] (2018) attributes by **co-occurrence**; [[c-obso]] (2022/23) by **deviation from a predicted reference**. Neither cites the other, and they are directly comparable on a single match — an unrun test.^[generated: rests-on: absence:no-held-source-compares-sgg-and-cobso — ⚠️ re-checked 2026-08-07: still holds. Nakahara et al. compare against C-OBSO and OBSO, not SGG.] Both independently find that **occupation and generation are distinct skills**, in different leagues. See [[space-creation]].

## The Off-Ball Metrics Have Now Been Compared, and They Disagree

> **New section, 2026-08-07. The most consequential result of the Nakahara ingest, and it cuts against the direction of travel above.**

Six mechanisms had accumulated on the assumption that more coverage is better. The first head-to-head test of whether they measure the same thing:

| Nakahara $Q$ against | $\rho$ | $N$ |
|---|---|---|
| [[c-obso\|C-OBSO]] | **0.182** *(no relationship)* | 14 |
| [[obso\|OBSO]] | −0.305 | 14 |
| Season goals | −0.761 | 14 |
| Expert match ratings | −0.218 | 14 |

C-OBSO and the Q-values come from **the same research group, the same club, the same season, the same [[data-stadium|data provider]] and essentially the same 14 players.** Both are presented as measures of off-ball contribution. They are unrelated.

**The paper's reading is benign and partly self-defeating.** C-OBSO ranks forwards highly; Q-values rank midfielders and defenders highly; so they capture different aspects. Grant that, and it follows that **neither measures "off-ball contribution" as such** — each measures a positional slice of it, and the shared name is doing work the mathematics does not support. C-OBSO's construction says as much: shot-ending sequences only, credit for improving a *shooter's* chance, negatives clipped.

Two things follow for this synthesis.

**First, the reliability gap and the disagreement gap are the same gap.** If two metrics disagree, either they measure different constructs or at least one is unstable — and with no reliability figure for any off-ball metric, those cannot be told apart. That makes a split-half estimate the cheapest informative measurement in this entire area, ahead of any seventh mechanism.

**Second, `no-cross-framework-benchmarking` is weakened but survives.** Nakahara et al. compare against [[obso|OBSO]] — a [[william-spearman|Spearman]] metric from an unrelated lineage — which is the first cross-lineage results comparison in the corpus. The weakening is partial: OBSO is used via the group's own modified reimplementation, C-OBSO is their own predecessor, and $N = 14$.^[generated: rests-on: absence:no-held-source-benchmarks-across-frameworks, source:gvdep-vs-vdep-comparison, source:nakahara-obso-cobso-correlations — ⚠️ **weakened twice**: at GVDEP (same-group, own-predecessor) and at Nakahara (cross-lineage, but reimplemented and tiny sample). Re-check on any ingest reporting two frameworks' outputs on one dataset.]

> ### `no-cross-framework-benchmarking`
> **Comparison happens within research lines and, where it now crosses them, is done by one line on its own reimplementation of the other.**

Groups still do not benchmark against a competitor's published numbers. This is why the tables above compare **design characteristics rather than results**.

Note the contrast with the vault's computer-vision material, where [[camera-calibration-benchmarking|ProCC]] exists precisely to make cross-method comparison possible and consolidates a leaderboard across groups. **The benchmarking gap is a property of one research community, not of sports analytics as a field.**

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning.^[generated: declared on [[representation-learning]]; never tested against a case it was not built to fit. **Checked 2026-08-07:** Nakahara et al. are a candidate fourth case — raw positions and velocities into a [[gated-recurrent-unit|GRU]], no pitch-control model, no geometric features, on only 1,669 training sequences. The rule predicts this should fail; the paper reports no accuracy metric against which failure could be observed, so the case is *uninformative* rather than confirming or falsifying. rests-on: claim:handcrafted-features-rule] A working heuristic, not a finding.

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]): OBSO 0.26 against next-match goals, shots 0.17, goals 0.12. **Team level**: LPV 0.28, HPUS 0.26, xG 0.17, goals 0.11. **Goals are the worst predictor of future goals in both.**

Six validation modes, ascending: self-prediction → cross-horizon consistency → **[[construct-validity|agreement or divergence against other metrics]]** → external outcome → external criterion outside the pipeline ([[c-obso]] vs salary) → **validation of a component against a directly observable quantity** ([[pass-probability-model|PPCF]] against 5,471 held-out pass receivers).

The third is inserted 2026-08-07 and belongs low in the order for a reason. Nakahara et al. rest entirely on it, and every comparison they report is a *divergence* read as favourable.

> ### `discriminant-claims-need-a-convergent-anchor`
> **A metric validated only by divergence from existing measures cannot be distinguished from a metric measuring nothing. Noise also diverges from goals.**
> ^[generated: declared on [[construct-validity]]. rests-on: source:nakahara-negative-goal-correlation, source:nakahara-no-ground-truth]

The sixth mode is rarely available, and its absence explains the rest: almost nothing here has an observable ground truth. [[wide-open-spaces-space-creation|Fernández & Bornn]] state it plainly — *"there is no existance of ground truth data regarding the quantification of spaces in soccer"* — and validate by expert video review instead. Nakahara et al. say the same of a Q-value and reach for correlations.

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by six off-ball mechanisms. **Errors of omission** are addressed in principle by route 6 — every available action carries a value, so a valuable run *not* made is visible — but no source reports it.
3. **Individual defensive credit is computed but not reported.** [[drso|DRSO]] computes it per named defender; every published result averages to teams. One aggregation step, not a missing method. Nakahara et al. are the same story for prescription: computed, not reported.
4. **No ground truth** for most quantities — hence the substitute tests, and hence how much weight the one directly validated component carries.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models.** [[c-obso]] predicts 3 of 22; [[space-occupation-gain|SOG/SGG]] analyse one match; Nakahara et al. use 14 players from one club, attacking third only, with independent agents that do not model each other.
7. **Component-level divergence, invisible in framework-level comparison.** Two [[pitch-control-traditions-compared|pitch-control traditions]], four [[shot-value-formulations-compared|shot-value formulations]], [[tracking-error-propagation|tracking error]] nobody propagates, two uncompared space-creation methods, and PPCF parameters two papers attribute to the wrong Spearman paper.

   The pitch-control case is now **structurally explained**: neither tradition cites the other, and both position against [[voronoi-tessellation]]. They are **siblings framed against a common ancestor, not rivals** — which is why the comparison is nobody's responsibility and has stayed undone for eight years.

8. **Strategy-space coarsening** is the price of prescription — and, per [[action-space-design]], the coarsening also fixes what "suboptimal" can mean, so frameworks with different action spaces are not measuring one construct even when they report one quantity.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent** — weakened twice, above.
11. ⚠️ **Where metrics have been compared, they disagree.** Newly the field's most pressing problem, and it displaces "not enough mechanisms".

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression]] to shortlist; [[scoutgpt|simulation]] for fit.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two with external validation. ⚠️ They disagree with the RL Q-values, so treat any one of the three as a *view* rather than a measurement.
- **Identifying a defender or deep midfielder whose output understates him** → the RL Q-values are the only metric here that ranks such players top, but on 14 players and with no reliability figure.
- **Valuing movement rather than position** → [[space-occupation-gain|SOG]] or the RL Q-values, the two rate metrics.
- **Coaching a decision** → [[xsot|SPC]]. **A position** → [[drso|DRSO]]. **A pass** → [[physics-based-pass-probabilities|hypothetical passing]], if you can afford the search.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]]. Team level.
- **Working from broadcast video rather than a tracking licence** → [[gvdep|GVDEP]], [[drso|DRSO]], [[obso|OBSO]]. Not the RL route, which needs full tracking.
- **Small-data modelling** → [[theory-based-modelling|theory-based features]]; avoid tree ensembles.
- **Any thresholded classifier** → tune the cutoff. See [[class-imbalance-evaluation]].

## Open Questions

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[action-supervision]] · [[action-space-design]] · [[construct-validity]]
- [[game-theory]] · [[xsot]] · [[drso]] · [[theory-based-modelling]] · [[policy-modelling]] · [[counterfactual-simulation]]
- [[obso]] · [[c-obso]] · [[space-occupation-gain]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]] · [[voronoi-tessellation]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]] · [[pass-probability-model]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]] · [[representation-learning]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]] · [[recruitment]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[hiroshi-nakahara]]
