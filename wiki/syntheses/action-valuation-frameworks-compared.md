---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
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

*(Formerly "Action Valuation Frameworks Compared" — renamed as the vault outgrew the valuation-only framing.)*

> **On provenance.** A synthesis generates claims that no single source states. The load-bearing ones are marked `^[generated]` at their point of use, with the fuller argument — and what would falsify it — on the linked concept page. Claims marked `absence:` have a built-in expiry date: they hold only until a source is acquired that contradicts them, which is how most of this vault's corrections have arrived. Eight open questions have their own pages, listed at the foot.

## The Six Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[martingale-epv]], [[pass-carry-reward\|PCR]], [[expected-value-possession-framework\|Fernández et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision | [[xsot\|Yeung & Fujii]] |

**Prescription is genuinely a sixth task.**^[generated: the six-task division is this vault's organising scheme; no source proposes it. rests-on: absence:no-source-proposes-a-task-taxonomy — ⚠️ expires if a survey paper is ingested]

## Task 1: Valuation

All frameworks instantiate one equation:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

| | [[expected-threat\|xT]] | [[vaep]] | [[martingale-epv\|Mart. EPV]] | [[pass-carry-reward\|Shelopugin]] | [[expected-value-possession-framework\|Fernández]] | [[vdep]] | [[gvdep]] | [[obso]] | [[c-obso]] | [[xsot\|xSOT]] |
|---|---|---|---|---|---|---|---|---|---|---|
| **Perspective** | Attack | Attack | Attack | Attack | Attack | **Defence** | **Defence** | Attack | Attack | **Both** |
| **Whose value** | Actor | Actor | Actor | Actor | Actor | Team | Team | Receiver | **Teammate's** | Actor |
| **Target** | Goal | Goal | Points | Decayed xG | Next goal | **Recovery** | **Recovery** | Goal | Goal | **Shot on target** |
| **On/off ball** | On | On | On | On | Both | Off (team) | Off (team) | **Off** | **Off** | Both |
| **Mechanism** | DP | Boosting | Bayesian process | Boosting ×9 | Neural ×9 | XGBoost ×2 | XGBoost ×4 | **Physical** | **Counterfactual** | **[[theory-based-modelling\|Hybrid]] + game** |
| **Weighting** | — | — | — | — | — | Frequency ratio | **VAEP score scale** | — | — | — |
| **[[interpretability]]** | **High** | Low | Low | Low | Moderate | Moderate | Moderate | **High** | Moderate | Moderate |
| **[[split-half-reliability\|Reliability]]** | **0.89** | 0.25 | — | — | N/A | N/A | N/A | — | — | — |
| **Cost** | Trivial | Modest | **461 CPUs** | Modest | Modest | Modest | **Low** | **Low** | High | Low |

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]] is the exception: tracking-based, off-ball, highly interpretable, at low cost, because it is **physical rather than learned**.

**Six of ten produce no usable player season rating.** The frameworks most often called state-of-the-art are the least usable for [[recruitment]].

⚠️ The reliability row is thinner evidence than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

The dashes in that row are themselves a finding:

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric in this vault has a reported split-half reliability.^[generated: an absence claim, established by checking every such framework held here. rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ expires on ingest of any off-ball or defensive valuation paper. Re-checked on the GVDEP ingest, 2026-07-27: still holds. Also on [[off-ball-value]] and [[defensive-valuation]].] Since reliability is the criterion that matters most for [[recruitment]], the metrics best suited to finding undervalued players are the ones whose stability is least known.

### Axis 1: Perspective — attacking or defending

[[football-defence-evaluation-vdep|Toda et al.]] report VAEP's conceding classifier at **F1 = 0.000** on 45 matches.

⚠️ That figure is near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. [[gvdep|GVDEP]] reports 0.08–0.15 on a different dataset at a comparable base rate, confirming the zero is **not a fixed property of the model**. The conclusion may still be right — VAEP correlates $\approx 0$ with goals conceded independently — while the diagnostic is wrong. See [[vaep-conceding-classifier]].

> ### `offensive-bias-four-causes`
>
> **Offensive bias has four distinct causes with four different remedies:** definitional (change the target), data (tracking), modelling choice (model duels), and statistical (a frequent proxy).
>
> ^[generated: constructed in this vault; no source enumerates these. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question per the caveat above, so the fourth cause is the least secure. Declared on [[action-valuation]].]

### Axis 2: Target rarity

See [[rare-event-proxy-targets]]. Five frameworks substitute a denser proxy for goals. **The proxy is not a neutral substitution; it reorganises the model around itself** — [[c-obso|C-OBSO]] weights the goalkeeper *double*, [[xsot|Yeung & Fujii]] remove him entirely, because a save still counts as *on target*.

### Axis 3: Whose value — actor, receiver, or beneficiary

Three distinct answers where the vault long had one. [[c-obso]] correlates 0.45 with salary on players where [[obso|OBSO]] (−0.28) and goals (−0.23) do not. See [[space-creation]].

### Axis 4: Observed policy or optimal policy

Every framework except [[xsot|Yeung & Fujii]] estimates value under the **observed** policy, on the stated ground that the optimal-policy counterfactual is unfounded.

[[game-theory|Yeung & Fujii]] show when that objection bites: restrict to a **two-action game** and payoffs for unobserved profiles become *estimable* rather than extrapolated. **The barrier is the size of the action space, not the observational nature of the data.** See [[policy-modelling]].

### Axes 5–7: intent vs outcome, attributable possession, realised vs available

Whether the model sees how the action turned out ([[intent-vs-outcome-valuation]]); whether contested events are visible ([[symmetrical-duel-valuation]]); whether unrealised options can be valued ([[probability-surface|pass surfaces]]).

### Credit assignment and free parameters

Six horizon positions, from [[vaep]]'s fixed $k=10$ window through Fernández et al.'s hard 15 s cutoff to [[temporal-discounting|Shelopugin's]] geometric decay and [[obso|OBSO's]] single next-event horizon.

**Four frameworks still carry an asserted free parameter with no sensitivity analysis** — $\gamma$, $\epsilon$, $k$, and C-OBSO's 4 s.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ narrowed 2026-07-27; GVDEP sweeps n_nearest and supersedes VDEP's C, so the surviving absence concerns horizon and weighting parameters only. Declared on [[model-selection]].]

**VDEP's $C$ has been superseded**, not merely questioned: [[gvdep|GVDEP]] replaces the frequency-derived constant with VAEP-derived score-scaled weights. That is the vault's only instance of an asserted parameter being fixed by principled derivation.

[[obso|Spearman]] remains the exception on the others, because `physical-units-admit-priors`^[generated: declared on [[model-selection]]; rests on a single source's parameter table and is the most fragile of the vault's multi-page claims by dependency alone]. See [[free-parameters-load-bearing]].

## Observed Versus Optimal: What the Two Results Do and Do Not Share

| Source | Method | Observed | Best available | Gap |
|---|---|---|---|---|
| [[expected-value-possession-framework\|Fernández et al.]] | Pass-value surface | 0.032 | 0.112 | 0.080 |
| [[xsot\|Yeung & Fujii]] | Game-theoretic payoffs | 0.0866 (shoot) | 0.2456 (pass) | 0.159 |

**Correction, 2026-07-27.** An earlier revision described the two as "locating the divergence in the same place." That was too strong:

- **Fernández et al.** measure suboptimal **targeting *within* an action**.
- **Yeung & Fujii** measure suboptimal **selection *between* actions**.

So **"shooters shoot too much" is Yeung & Fujii's claim alone.** What the two jointly support is convergence on the **existence** of a gap, not its cause — and at least three explanations produce a large gap with no decision error present: average-player models applied to specific players, unmodelled execution difficulty, and an assumed-rational opponent.

Neither paper tests whether the gap predicts anything outside its own model.^[generated: rests-on: absence:neither-paper-validates-the-gap] See [[observed-versus-optimal-decisions]].

## Off-Ball Valuation: Four Mechanisms

A player has the ball for roughly **3 of 90 minutes**.

| | Surface at position | 22 positions in state | Predicted reference | Physical surface |
|---|---|---|---|---|
| Values | The receiver | The defence | The creator | The receiver |
| Output unit | Player | **Team only** | Player | Player |
| Example | EPV surface | [[vdep]], [[gvdep]] | [[c-obso]] | [[obso]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]], where a Shapley-style objection demotes it from law to tendency. rests-on: claim:counterfactual-individuates] VDEP and C-OBSO use comparable tracking data; VDEP produces one number per configuration, C-OBSO intervenes on one *named* player.

[[gvdep|GVDEP]] adds a bound on how much data the second column needs: **ball-gain prediction saturates at three or four players**, and scores, concedes and being-attacked gain nothing from player positions at all. Most of what VDEP fed its classifier was unnecessary.

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Predicts | Next event | Event + time | Event + exact $(x,y)$ | Event + lineup | **All agents' positions** |
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**Forecasting produces metrics as a by-product**, needing no outcome labels, so goal sparsity never bites. See [[event-prediction]].

### The handcrafted-features question

Seq2Event degrades *without* engineered geometry; [[sig-model]] degrades *with* it; [[xsot|Yeung & Fujii]] find a [[theory-based-modelling|theory-based feature]] essential and raw coordinates actively harmful.

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning; encode nothing else.^[generated: declared on [[representation-learning]]. A reconciliation constructed here; none of the three sources addresses the others, and it has never been tested against a case it was not built to fit. rests-on: claim:handcrafted-features-rule] Treat it as a working heuristic, not a finding.

GVDEP supplies a fourth data point consistent with it: its concedes classifier gets **worse** as player positions are added (F1 0.15 → 0.08), with only 186 positives. The same sample-size logic explains why tree ensembles **won** on VAEP's 8.5M actions and came **last** on Yeung & Fujii's 2,575 shots. See [[gradient-boosting]].

## Tasks 3–5: Clustering, Counterfactual, Tactical

[[football-event-sequences-point-process-mixture|Possession clustering]] into tactical types, validated by [[adjusted-rand-index|ARI]] and BIC. [[scoutgpt|Generative simulation]] against [[transfer-performance-prediction|regression on context]] — regression to narrow a market, simulation to discriminate among survivors. And [[tactical-analysis|pressing analysis]], which has the **weakest validation in the vault** and unusually persuasive output.

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]):

| Predictor | Next-match goals |
|---|---|
| **[[obso\|OBSO]]** | **0.26** |
| Shots | 0.17 |
| Goals | 0.12 |

**Team level** ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]): LPV 0.28 and HPUS 0.26 against next-match goals, versus xG 0.17 and goals 0.11.

**Goals are the worst predictor of future goals in both.** A third instance: [[xsot|xSOT]] correlates 0.58 with average goals where [[expected-goals|xG]] manages 0.46.

Spearman's remains the strongest — player-level, against an independent outcome, and beating the outcome's own lagged value. Contrast [[epv-control-duel-skills-football|Shelopugin's]] next-season forecast, which predicts the metric's own future value.

Four validation checks in ascending strength: self-prediction → cross-horizon consistency ([[vdep]]) → external outcome (Spearman) → external criterion outside the pipeline ([[c-obso]] vs salary).

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies, with the caveat above.
2. **On-ball bias, substantially narrowed** by four off-ball mechanisms. Still uncovered: **errors of omission**, movement beyond short windows.
3. **Individual defensive credit** — addressed in the literature (Umemoto & Fujii, **2023**), not held here. ⚠️ Note this is a *different paper* from the 2022 GVDEP work now held, which is team-level. See [[defensive-valuation]].
4. **No ground truth**, which is why reliability, predictive validity, calibration, imbalance-robust metrics and external criteria have all become substitute tests — and why two of them turn out not to be independent.
5. **[[selection-bias]] throughout.**
6. **Scale limits on interaction models** — [[c-obso]] predicts 3 of 22 players. GVDEP suggests this may be less binding than assumed: if three or four players suffice, full-squad evaluation may be unnecessary rather than merely expensive.
7. **Component-level divergence, invisible in framework-level comparison.** Two uncompared [[pitch-control-traditions-compared|pitch-control traditions]]^[generated: `pitch-control-traditions-uncompared`, declared on [[pitch-control]]], four unbenchmarked [[shot-value-formulations-compared|shot-value formulations]]^[generated: `shot-value-formulations-unbenchmarked`, declared on [[expected-goals]]], and [[tracking-error-propagation|tracking error]] that nobody propagates^[generated: `no-tracking-uncertainty-propagation`, declared on [[multi-object-tracking]]; weakened 2026-07-27, since GVDEP measures the cost of incomplete observation though not of positional error].
8. **Strategy-space coarsening** is the price of prescription.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent**, below.

> ### `no-cross-framework-benchmarking`
>
> **Almost no framework in this vault has been benchmarked against another on a shared task.**
>
> ^[generated: the vault's most-repeated absence claim, and the basis of the argument that its comparison tables must rest on design characteristics rather than results. rests-on: absence:no-held-source-benchmarks-across-frameworks, source:gvdep-vs-vdep-comparison — ⚠️ **weakened 2026-07-27**, and the single highest-value absence claim to re-check on every ingest.]

> **Weakened, 2026-07-27.** [[gvdep|GVDEP]] compares itself directly against [[vdep|VDEP]] on identical data, reporting both definitions side by side. That is a genuine like-for-like comparison and the first in the vault.
>
> Two qualifications keep the claim alive in weakened form. It is **same-group, own-predecessor** work — the authors were replacing their own method, not surveying alternatives, which is the internal-justification pattern rather than a benchmark. And it is **within one lineage**: nothing compares VDEP or GVDEP against [[vaep]]'s defensive half, [[c-obso]], or any framework outside the Fujii group.
>
> So the accurate statement is now: **comparison happens within research lines and never across them.** That is a weaker and more interesting claim than blanket absence, because it identifies the boundary — groups benchmark against themselves, and nobody benchmarks against a competitor.

This is why the tables above compare **design characteristics rather than results** — a choice forced by the literature, not preferred.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression on club/league strength]] to shortlist; [[scoutgpt|simulation]] for fit; [[player-development-curve|PDC]] for trajectory.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two metrics here with external validation.
- **Coaching a decision, not describing one** → [[xsot|the SPC framework]], with the caveat above.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]] — the same approach with a principled weighting and no full-tracking requirement. Team level only.
- **Live or post-match decision support** → [[expected-value-possession-framework|Fernández et al.]]; [[obso|OBSO]] for ranking moments at far lower cost.
- **Valuing what was available** → [[probability-surface|pass surfaces]].
- **Separating decision quality from finishing** → [[intent-vs-outcome-valuation|I-VAEP vs O-VAEP]].
- **Aerial or physical targets** → [[duel-skill-rating]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]] over raw inputs, and avoid tree ensembles.
- **Working from broadcast video rather than a tracking licence** → [[gvdep|GVDEP]] and [[obso|OBSO]], both designed to minimise data requirements.
- **Shot quality alone** → xG, a *component* of the others rather than a competitor.

## Open Questions

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[game-theory]] · [[xsot]] · [[theory-based-modelling]] · [[policy-modelling]] · [[reinforcement-learning]]
- [[obso]] · [[c-obso]] · [[space-creation]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[representation-learning]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[event-prediction]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]] · [[player-rating-time-series]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]]
- [[william-spearman]] · [[keisuke-fujii]] · [[calvin-yeung]] · [[javier-fernandez]] · [[rikuhei-umemoto]]
