---
title: "Football Modelling Tasks Compared"
type: synthesis
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, evaluation, counterfactual, game-theory, clustering, event-prediction, reliability, predictive-validity, construct-validity, time-series, recruitment, transfer-prediction, duel-analysis, discounting, selection-bias, probability-surface, tactical-analysis, model-decomposition, proxy-target, class-imbalance, trajectory-prediction, pitch-control, theory-based-modelling, reinforcement-learning, multi-agent, action-space, simulator, domain-adaptation]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/understanding_football_posessions_using_path_signatures.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/wide_open_spaces_creation_football.md, raw/papers/beyond_expected_goals.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.9
provenance:
  extracted: 45%
  inferred: 34%
  generated: 16%
  imported: 2%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-08-07
---

# Football Modelling Tasks Compared

The vault's football-analytics sources are easily mistaken for variations on one problem. They are not. They divide into **seven distinct tasks**, each answering a different question, with different data requirements and validation strategies.

> **On provenance.** A synthesis generates claims that no single source states. The load-bearing ones are marked `^[generated]` at their point of use, with the fuller argument — and what would falsify it — on the linked concept page. Claims marked `absence:` have a built-in expiry date. Eight open questions have their own pages, listed at the foot.

## The Seven Tasks

| Task | Question | Unit | Examples |
|---|---|---|---|
| **Valuation** | How good was that action or position? | Action → player or team | [[expected-goals\|xG]], [[expected-threat\|xT]], [[vaep]], [[vdep]], [[gvdep]], [[obso]], [[c-obso]], [[space-occupation-gain\|SOG]], [[martingale-epv]], [[expected-value-possession-framework\|Fernández et al.]], [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |
| **Forecasting** | What happens next? | Event or trajectory | [[seq2event]], [[nmstpp]], [[sig-model]], [[trajectory-prediction\|GVRNN]] |
| **Clustering** | What kind of sequence is this? | Possession | [[football-event-sequences-point-process-mixture\|Mixture model]] |
| **Counterfactual / transfer** | What if this player joined? | Episode or season | [[scoutgpt]], [[transfer-performance-prediction\|Shelopugin regression]] |
| **Tactical** | How does this team play, and how do we counter it? | Team configuration | [[tactical-analysis\|Pressing analysis]], possession clustering |
| **Prescription** | What *should* the player have done? | Decision or position | [[physics-based-pass-probabilities\|Spearman et al. (2017)]], [[xsot\|Yeung & Fujii]], [[drso\|DRSO]] |
| **Simulation** | Can we reproduce real football in a synthetic environment? | A whole episode | **[[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]]** |

**The seventh is new as of 2026-08-07**^[generated: the task division is this vault's organising scheme; no source proposes it, and no source describes simulation as a distinct task from counterfactual valuation. rests-on: absence:no-source-proposes-a-task-taxonomy] and is genuinely distinct from counterfactual/transfer, though both generate. The difference is the **deliverable**: counterfactual work generates in order to produce a number about a player; simulation work generates in order to produce a faithful environment. [[scoutgpt|ScoutGPT]] would be a success if its player valuations were right even with implausible rollouts; Fujii et al. would not.

That distinction turns out to matter, because the two tasks succeed and fail differently — see "Where Generation Breaks" below.

**Prescription remains the oldest task, not the newest.**

| Instance | Year | Optimises over | Choice set |
|---|---|---|---|
| **[[physics-based-pass-probabilities\|Hypothetical passing]]** | **2017** | Ball velocity vector | Continuous, searched by annealing |
| [[drso\|DRSO]] | 2023 | Defender position | Four grid vertices |
| [[xsot\|SPC framework]] | 2024 | Action | Two: shoot or pass |
| *([[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] — **could**, and does not)* | 2023 | Action | **14, per player, per timestep** |

The 2017 instance is the only one that **did not scale**. Both successors solved that by **coarsening the choice set** rather than by cheaper search. **You can only prescribe over a choice set you can enumerate** — but Nakahara et al. show enumerability is not sufficient: they hold the largest tractable prescriptive choice set in the vault, have the observed-versus-maximum gap one subtraction from their output tensor, and report correlations with season goals instead. **A group also has to want to make the prescriptive claim.** See [[observed-versus-optimal-decisions]].

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

**The central trade-off** — richer state buys sensitivity and pays in stability, interpretability and cost — holds across most of the table. [[obso|OBSO]], [[space-occupation-gain|SOG]] and [[drso|DRSO]] are the exceptions: tracking-based, interpretable and cheap, because all three are **physical or geometric rather than learned**.

### A fifth estimation style

Van Roy et al.'s taxonomy has three styles; this vault added *counterfactual* as a fourth and **reinforcement-learning-based** as a fifth: learn $Q(s,a)$ over an explicit [[action-space-design|action space]] by bootstrapping against reward.^[generated: added on [[action-valuation]]; distinct from *action-based* because value is learned against reward by [[temporal-difference-learning|bootstrapping]] rather than regressed against an outcome label, and from *counterfactual* because no reference baseline is differenced. rests-on: source:vanroy-three-style-taxonomy, source:nakahara-sarsa-td]

⚠️ Reliability evidence is thinner than it looks. [[split-half-reliability]] and [[performance-volatility]] measure **the same variance component** with opposite interpretations. See [[within-season-variation-noise-or-signal]].

> **`no-reliability-for-off-ball-metrics`** — no off-ball or defensive metric here has a reported split-half reliability.^[generated: rests-on: absence:no-held-source-reports-off-ball-reliability — ⚠️ re-checked on the GVDEP, DRSO, Spearman-2017, Wide Open Spaces, Nakahara and **Fujii** ingests: still holds. **Six mechanisms, zero reliability estimates.**]

### Axis 1: Perspective

VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15, and [[physics-based-pass-probabilities|Spearman et al.]] demonstrate the mechanism directly. See [[vaep-conceding-classifier]].

Two structural alternatives to a rarity-stricken classifier now exist, both from the Fujii group and both undiscussed by it. Nakahara et al. put conceding in the **reward** at possession level, diluting the base rate. Fujii et al. add a **shot reward of +1** explicitly "because the goal reward was sparse and limited" — [[rare-event-proxy-targets|proxy substitution]] executed inside a reward function rather than a classifier target. **The group's signature move has migrated into RL without anyone noting the migration.**

> ### `offensive-bias-four-causes`
> **Four distinct causes with four different remedies:** definitional, data, modelling choice, statistical.
> ^[generated: rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero — the fourth premise is under question. Declared on [[action-valuation]].]

### Axes 2–5

**Target rarity** — the proxy reorganises the model around itself, now including reward functions. **Whose value** — six answers. **Observed or optimal policy** — see prescription. **[[action-space-design|Action space]]** — arguably the most consequential, because it fixes which counterfactuals a framework can pose.

### Free parameters

**Sixteen asserted parameters carry no sensitivity analysis**, up from four across five ingests, and of **six kinds**.^[generated: rests-on: absence:no-sensitivity-analysis-on-horizon-parameters — ⚠️ re-checked at the Fujii ingest, which adds two. Still holds. Declared on [[model-selection]].]

Horizons are likely self-limiting; $\gamma$ is a shape parameter spanning two orders of magnitude; SOG/SGG's $\delta$, $\alpha$ and Nakahara's velocity thresholds are **gates**; $\lambda_1$ is a **prior strength**; and — new — the **training-step count** is a stopping parameter.

> ### `where-you-stop-is-a-modelling-choice`
> **When imitation and reward objectives trade off during training rather than only across hyperparameter settings, the training-step count is a free parameter with the same status as a loss weight — and it is almost never reported as one.**
> ^[generated: declared on [[imitation-reward-tradeoff]]. rests-on: source:fujii-training-step-ordering]

⚠️ **The vault predicted where the $\lambda_1$ answer would be found and was wrong.** arXiv:2305.13030 was acquired and reports **no value for $\lambda_1$ at all** — the methods paper is less specific about its own mechanism than the applied paper citing it. The recorded lesson: **methods papers optimise for ablation questions ("is this component needed?"), not calibration questions ("how much of it?")**. See [[free-parameters-load-bearing]].

## Off-Ball Valuation: Six Mechanisms

A player has the ball for roughly **3 of 90 minutes**.

| | Surface at position | Control × value | Positions in state | Predicted reference | Optimal position | **Learned $Q$** |
|---|---|---|---|---|---|---|
| Values | The **receiver** | The **occupier** | The **defence** | The **creator** | The **defender** | **The mover** |
| Measures | A level | **A rate** | A level | A rate | A level | **A rate** |
| Needs a baseline | Yes | Yes | No | **Yes** | **Yes** | **No** |
| Reported unit | Player | Player | **Team only** | Player | **Team** | **Player** |
| Example | EPV surface, [[obso]] | [[space-occupation-gain\|SOG]] | [[vdep]], [[gvdep]] | [[c-obso]] | [[drso]] | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]]. **Weakened**: route 6 individuates by *agent decomposition*, a second non-counterfactual individuator alongside SGG's spatial predicate. rests-on: claim:counterfactual-individuates]

**Space creation is covered twice, by unrelated mechanisms**, never compared.^[generated: rests-on: absence:no-held-source-compares-sgg-and-cobso — ⚠️ re-checked 2026-08-07: still holds.] See [[space-creation]].

## The Off-Ball Metrics Have Been Compared, and They Disagree

| Nakahara $Q$ against | $\rho$ | $N$ |
|---|---|---|
| [[c-obso\|C-OBSO]] | **0.182** *(no relationship)* | 14 |
| [[obso\|OBSO]] | −0.305 | 14 |
| Season goals | −0.761 | 14 |
| Expert match ratings | −0.218 | 14 |

C-OBSO and the Q-values come from the same group, club, season, [[data-stadium|provider]] and 14 players. Both are presented as off-ball contribution. They are unrelated.

The paper's benign reading — different aspects — **concedes that neither measures "off-ball contribution" as such.**

**A further explanation surfaced with the Fujii ingest.** The four sources on this dataset each **subset it differently**: C-OBSO takes shot-ending sequences only (412), Nakahara et al. attacking-third possessions regardless of outcome, Fujii et al. last-pass sequences, VDEP everything. **Three overlapping but non-identical populations of football moments, from one dataset, in one group.** Metrics computed over different moments need not agree, and the shared name conceals it. See [[data-stadium]] and [[construct-validity]].

**First, the reliability gap and the disagreement gap are the same gap.** Without a reliability figure, "different constructs" and "one is unstable" cannot be separated. A split-half estimate is the cheapest informative measurement in this area, ahead of any seventh mechanism.

**Second, `no-cross-framework-benchmarking` is weakened but survives — and has been undermined from a new direction.** Nakahara et al. compare against [[obso|OBSO]], a Spearman metric from an unrelated lineage. But Fujii et al. **replace the shared simulator with a bespoke one**:

> ### `bespoke-environments-foreclose-comparison`
> **A research group that builds its own simulator to fix a shared one's shortcomings trades external comparability for internal control, and the trade is rarely acknowledged as a cost.**
> ^[generated: declared on [[nfootball]]. rests-on: source:fujii-nfootball-motivation, absence:no-held-source-benchmarks-across-frameworks]

[[google-research-football|GFootball's]] value was never its physics — it was that everyone used it. **Four papers now share the Data Stadium dataset and only one pair has been compared; the other five pairings are equally computable and unrun.** Availability is not sufficient.

## Task 7: Simulation, and Where Generation Breaks

> **New section, 2026-08-07.**

[[adaptive-action-supervision-multi-agent-rl|Fujii et al.]] attempt the forward approach: build an environment, learn inside it, reproduce real football. **It does not work.** DQAAS learned to pass and shoot without moving toward goal; plain DQN moved toward goal without passing or shooting. The demonstration did both.

They then test whether the cause is algorithmic — decentralised against centralised MARL (CDS), classic against recent deep RL — and conclude it is not, attributing the failure to **"the domain-specific modeling and reality of the simulator"**.

That confirms, from a held source, a correction the vault made on inference: **the football-RL bottleneck is simulator fidelity, not algorithm choice.** See [[domain-adaptation]] and [[reinforcement-learning]].

It also raises an uncomfortable question about task 4. [[scoutgpt|ScoutGPT]] and [[eventgpt|EventGPT]] re-generate **event tokens**; Fujii et al. re-generate **continuous multi-agent movement**.

> ### `regeneration-fidelity-scales-with-representation-coarseness`
> **Re-generation counterfactuals succeed where the representation is coarse enough that the generative model's errors stay inside the discretisation, and fail where it is fine enough that they compound into physically wrong behaviour.**
> ^[generated: declared on [[counterfactual-simulation]]. rests-on: source:scoutgpt-transfer-mae, source:fujii-football-reproducibility-failure]

**ScoutGPT's counterfactuals may work partly because event tokens hide the physics** — a generated {pass, carry, shot} sequence cannot be physically impossible in the way a generated trajectory can. Whether that makes the counterfactual more trustworthy or merely less falsifiable is untested.

## Task 2: Forecasting

| | [[seq2event]] | [[nmstpp]] | [[sig-model]] | [[scoutgpt]] | [[trajectory-prediction\|GVRNN]] |
|---|---|---|---|---|---|
| Handcrafted features | **Required** | Used | **Harmful** | Minimal | None |
| Derived metric | poss-util | [[hpus]] | [[lpv]] | Simulated VAEP | **[[c-obso]]** |

**`handcrafted-features-rule`** — encode structure the representation cannot recover *and* the data cannot support learning.^[generated: declared on [[representation-learning]]. **Checked twice, 2026-08-07:** Nakahara et al. (raw positions into a GRU, 1,669 sequences) and Fujii et al. (raw state into a DQN, 1,121 sequences) are both candidate fourth cases. Neither reports an accuracy metric against which the rule's prediction of failure could be observed — Fujii et al. report reward and DTW distance instead. Both cases **uninformative** rather than confirming or falsifying. rests-on: claim:handcrafted-features-rule]

Two candidate tests arriving and both being uninformative is itself worth noting: **the rule may be unfalsifiable within this literature**, because the papers that would test it do not report the quantity it predicts.

## Metrics Beat Outcomes at Predicting Outcomes

**Player level** ([[beyond-expected-goals|Spearman, 2018]]): OBSO 0.26 against next-match goals, shots 0.17, goals 0.12. **Team level**: LPV 0.28, HPUS 0.26, xG 0.17, goals 0.11. **Goals are the worst predictor of future goals in both.**

Six validation modes, ascending: self-prediction → cross-horizon consistency → **[[construct-validity|agreement or divergence against other metrics]]** → external outcome → external criterion outside the pipeline ([[c-obso]] vs salary) → **validation of a component against a directly observable quantity** ([[pass-probability-model|PPCF]] against 5,471 held-out pass receivers).

> ### `discriminant-claims-need-a-convergent-anchor`
> **A metric validated only by divergence from existing measures cannot be distinguished from a metric measuring nothing. Noise also diverges from goals.**
> ^[generated: declared on [[construct-validity]]. rests-on: source:nakahara-negative-goal-correlation, source:nakahara-no-ground-truth]

**Fujii et al. contribute the honest version of mode 1.** Their [[dynamic-time-warping|DTW]] distance to a held-out demonstration is self-to-self reconstruction, reported as a headline metric alongside reward — and it does not improve. Compare [[eventgpt|EventGPT]], whose simulated value for Saka *exceeds* ground truth and which then uses the simulated value as the baseline. **The same test, reported honestly in one place and worked around in the other.**

## Limitations Shared Across Tasks

1. **Offensive bias** — four causes, four remedies.
2. **On-ball bias, substantially narrowed** by six off-ball mechanisms. **Errors of omission** addressed in principle by route 6; unreported.
3. **Individual defensive credit is computed but not reported.** Same story for prescription in Nakahara et al.
4. **No ground truth** for most quantities.
5. **[[selection-bias]] throughout.**
6. **Scale limits.** [[c-obso]] predicts 3 of 22; [[space-occupation-gain|SOG/SGG]] analyse one match; Nakahara et al. use 14 players from one club; **Fujii et al. use 16 training and 5 test episodes and cannot reach 11v11.**
7. **Component-level divergence, invisible in framework-level comparison.** Two [[pitch-control-traditions-compared|pitch-control traditions]], four [[shot-value-formulations-compared|shot-value formulations]], [[tracking-error-propagation|tracking error]] nobody propagates, two uncompared space-creation methods, PPCF parameters attributed to the wrong paper — and now **two action spaces (14 vs 12) and two regularisers ($L_1$ vs $L_2$) chosen differently by overlapping authors on one dataset, neither citing the other's choice.**
8. **Strategy-space coarsening** is the price of prescription — and per [[action-space-design]], the coarsening fixes what "suboptimal" can mean.
9. **Price is absent everywhere.**
10. **Cross-framework benchmarking is almost entirely absent** — weakened twice, and newly undermined by bespoke environments.
11. ⚠️ **Where metrics have been compared, they disagree.**
12. ⚠️ **Where the forward approach has been attempted, it failed** — and the cause is the environment, not the algorithm.

## Practical Guidance

- **Season-long recruitment** → xT for stability; [[transfer-performance-prediction|regression]] to shortlist; [[scoutgpt|simulation]] for fit, noting the coarseness caveat above.
- **Identifying an attacker whose output understates him** → [[obso|OBSO]] and [[c-obso]], the two with external validation. ⚠️ They disagree with the RL Q-values; treat any one as a *view*.
- **Identifying a defender or deep midfielder whose output understates him** → the RL Q-values, on 14 players and with no reliability figure.
- **Valuing movement rather than position** → [[space-occupation-gain|SOG]] or the RL Q-values.
- **Coaching a decision** → [[xsot|SPC]]. **A position** → [[drso|DRSO]]. **A pass** → [[physics-based-pass-probabilities|hypothetical passing]].
- **Simulating football** → nothing here works yet. [[nfootball|NFootball]] is the most recent attempt and its authors report the failure plainly.
- **Assessing a defence** → [[gvdep|GVDEP]] over [[vdep|VDEP]]. Team level.
- **Working from broadcast video** → [[gvdep|GVDEP]], [[drso|DRSO]], [[obso|OBSO]].
- **Small-data modelling** → [[theory-based-modelling|theory-based features]]; avoid tree ensembles.
- **Any thresholded classifier** → tune the cutoff. See [[class-imbalance-evaluation]].

## Open Questions

- [[pitch-control-traditions-compared]] · [[shot-value-formulations-compared]] · [[tracking-error-propagation]] — component-level gaps
- [[free-parameters-load-bearing]] · [[vaep-conceding-classifier]] — untested assumptions in held work
- [[within-season-variation-noise-or-signal]] · [[observed-versus-optimal-decisions]] · [[handcrafted-features-rule]] — claims this vault generated

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[expected-possession-value]] · [[tactical-analysis]]
- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[deep-q-network]] · [[action-supervision]] · [[action-space-design]] · [[construct-validity]]
- [[domain-adaptation]] · [[dynamic-time-warping]] · [[imitation-reward-tradeoff]] · [[nfootball]] · [[google-research-football]] · [[counterfactual-simulation]]
- [[game-theory]] · [[xsot]] · [[drso]] · [[theory-based-modelling]] · [[policy-modelling]] · [[imitation-learning]]
- [[obso]] · [[c-obso]] · [[space-occupation-gain]] · [[counterfactual-baseline]] · [[trajectory-prediction]] · [[pitch-control]] · [[voronoi-tessellation]]
- [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[martingale-epv]] · [[expected-goals]] · [[pass-carry-reward]] · [[pass-probability-model]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]] · [[model-selection]] · [[gradient-boosting]]
- [[hpus]] · [[lpv]] · [[sig-model]] · [[nmstpp]] · [[seq2event]] · [[scoutgpt]] · [[eventgpt]] · [[event-prediction]] · [[representation-learning]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[performance-volatility]] · [[recruitment]] · [[data-stadium]]
- [[william-spearman]] · [[javier-fernandez]] · [[luke-bornn]] · [[keisuke-fujii]] · [[rikuhei-umemoto]] · [[calvin-yeung]] · [[hiroshi-nakahara]] · [[atom-scott]]
