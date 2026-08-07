---
title: "Action Valuation"
type: concept
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, markov-model, evaluation, event-stream-data, time-series, recruitment, discounting, duel-analysis, probability-surface, proxy-target, counterfactual, theory-based-modelling, reinforcement-learning, temporal-difference, action-space, construct-validity]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/physics_based_pass_probabilities.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 19%
  generated: 8%
  imported: 0%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-08-07
---

# Action Valuation

Assigning a numeric value to each action a player performs, reflecting how much it improved or damaged their team's prospects. It exists because traditional statistics measure almost nothing: shots and assists together are **less than 1% of all on-the-ball actions**.

## The Unifying Equation

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]]: essentially all modern approaches share one form.

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

Frameworks differ only in **how they represent $S$** and **how they compute $Q$**. Deliberately reminiscent of [[reinforcement-learning]]: $Q$ is a state-value function, $V(a_i)$ an advantage.

> **Qualified 2026-08-07.** This page previously added "though almost none of this work is reinforcement learning, since nobody is learning a policy by interaction." That remains true of the frameworks below but is **no longer universally true** — [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] minimise a Bellman residual directly. See [[temporal-difference-learning]], where the distinction between *differencing a supervised model* and *bootstrapping* is set out.

## Five Styles

**Count-based** — weight each action type, sum counts. McHale & Scarf (2007); PlayeRank (2019). *No regard for context.*

**Possession-based** — value ball-progressing actions by how they change the goal chance *within a possession*. Mostly [[markov-game|Markov models]] solved by [[value-iteration]]. [[expected-threat|xT]] (2019). *Stops at the turnover, so cannot model risk.*

**Action-based** — rich features of action and context, framed as supervised learning. [[vaep]]; [[pass-carry-reward|Shelopugin]]; [[expected-value-possession-framework|Fernández et al.]]; [[vdep]]. *Loses [[interpretability]]; less stable.*

**Counterfactual** — value an action or position by comparison against a reference rather than a learned value function. [[c-obso]], [[drso]].^[generated: this fourth style is added here; Van Roy et al.'s taxonomy has three, and C-OBSO fits none of them. rests-on: source:vanroy-three-style-taxonomy, source:cobso-construction] See [[counterfactual-baseline]].

**Reinforcement-learning-based** — learn $Q(s,a)$ over an explicit [[action-space-design|action space]] by [[temporal-difference-learning|temporal difference]] against a reward, and read counterfactual values off the learned function.^[generated: a fifth style added 2026-08-07. Distinct from *action-based* because value is learned against reward by bootstrapping, not regressed against an outcome label; distinct from *counterfactual* because no reference baseline is differenced. rests-on: source:vanroy-three-style-taxonomy, source:nakahara-sarsa-td] [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]]. *Needs [[optical-tracking-data|tracking]]; values are unanchored and unvalidatable without external criteria.*

### A Physical Antecedent, Independently Derived

> **Added 2026-07-27.** Van Roy et al.'s taxonomy covers the learned traditions. It misses an earlier, physically-grounded one.

[[physics-based-pass-probabilities|Spearman et al. (2017)]] define pass value as

$$V_j = p_j\,f(x_{suc}) - (1 - p_j)\,f(x_{fail})$$

where $p_j$ is the modelled reception probability and $f$ is a state value — here "naïve", a negative exponential in distance to goal with three fitted constants.

This is the unifying equation in a different dress: **value is the probability-weighted difference between the state if the action succeeds and the state if it fails.** It appears **a year before [[vaep|VAEP]]** and from an entirely separate lineage.

Two things follow.

**The equation is more robust than its provenance suggests.** Two groups with different data, different methods and no shared citations arrive at the same algebraic form.

**The difference is where the probability comes from.** VAEP *learns* $P(\text{score})$ and $P(\text{concede})$ from labelled outcomes; Spearman *derives* $p_j$ from a physical model of interception and control. Same equation, opposite epistemology — see [[theory-based-modelling]].

Spearman's crude value function still produced metrics correlating 0.63–0.83 with shots and attacking-third passes, suggesting **the value function may matter less than the probability model feeding it** — the same question [[shot-value-formulations-compared]] raises about OBSO's weak score term.

## What Distinguishes the Approaches

| Axis | Options | Consequence |
|---|---|---|
| State representation | Zone → last-$k$ actions → tracking snapshot | Which actions can be valued |
| Horizon | Possession → next $k$ actions → next goal → unbounded | Whether risk is modelled |
| Estimation | DP → supervised → Bayesian process → [[structured-model-decomposition\|decomposed]] → counterfactual → physical → **[[temporal-difference-learning\|TD/RL]]** | Interpretability and cost |
| Data | [[event-stream-data]] → [[optical-tracking-data]] | Availability and off-ball coverage |
| **Outcome visibility** | Included → withheld | Execution or *decision* |
| **Credit assignment** | Fixed window → capped decay → hard cutoff → geometric decay → **flat, terminal reward only** | How value propagates back |
| **Possession attributability** | Assumed → modelled | Whether duels are visible |
| **Output granularity** | Per action → per location → **per (player, action, timestep)** | Whether *unrealised* options can be valued |
| **[[action-space-design\|Action space]]** | Implicit in the event stream → continuous surface → 4 profiles → 14 discrete | **What "a better decision" can mean** |
| **Perspective** | Attacking → defending | Whose success is measured |
| **Target rarity** | Goals → frequent proxies | Whether the classifier can learn |
| **Whose value** | The actor → a teammate's → the defender → **every player at once** | Whether space creation is credited |

**Credit assignment** now has five positions, with Nakahara et al.'s $\gamma = 1$ and terminal-only reward at the flat extreme.

**Action space** is new as an axis and is arguably the most consequential of them, because it fixes which counterfactuals a framework can even pose. See [[action-space-design]].

Every time-based approach carries a free parameter that no source subjects to sensitivity analysis. See [[free-parameters-load-bearing]] and [[model-selection]].

## Perspective: Attacking or Defending

Every framework above except [[vdep]] and [[gvdep]] measures **attacking** success and treats defence as its negative.

[[football-defence-evaluation-vdep|Toda et al.]] report VAEP's conceding classifier at **F1 = 0.000**. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. See [[vaep-conceding-classifier]].

The proposed fix is to change the target: predicting **ball recovery** and **being attacked** raises F1 to 0.522 and 0.484. See [[rare-event-proxy-targets]].

**A third position exists**, easily missed: [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] include conceding directly in the reward ($-1$ if the opponent scores immediately after the possession ends) rather than as a separate classifier. Because reward enters at the terminal frame of a sequence rather than as a per-event label, the rarity problem is diluted — one label per possession rather than per event. Whether that actually helps is untested; they report no defensive results.

## Whose Value: Actor or Beneficiary

Every framework except [[c-obso]], [[drso]] and [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] credits the player **performing** the valued act. C-OBSO credits the player whose movement improved *someone else's* chance; DRSO credits a defender for where he stood; Nakahara et al. credit **all ten attackers simultaneously and independently**, which sidesteps attribution rather than solving it — the ten Q-values need not sum to anything.

On the same 15 players, C-OBSO correlates 0.45 with annual salary while OBSO (−0.28) and goals (−0.23) do not. See [[space-creation]].

## The Metrics Do Not Agree With Each Other

> **Added 2026-08-07 — arguably the most important open problem on this page.**

The field's implicit assumption has been that more mechanisms means better coverage. The first head-to-head test of that assumption is not encouraging: [[c-obso|C-OBSO]] and Nakahara et al.'s Q-values, on the same club, season and data provider, correlate at **$\rho = 0.182$**.

Both are presented as measuring off-ball contribution. Neither reports [[split-half-reliability|reliability]], so "different constructs" and "one is unstable" cannot be separated.

This reframes the gap analysis below. **Adding a seventh mechanism is worth less than measuring whether the existing six agree.** See [[construct-validity]] and [[off-ball-value]].

## The Possession-Attributability Assumption

The equation needs to know *whose* prospects $Q$ describes. Obvious for a pass, undefined for an aerial duel. Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it.

## Valuing What Happened vs What Was Available

A full [[probability-surface|surface]] over pass destinations changes the question to "how good was that pass *relative to what was available*?" Realised 0.032 against best-available 0.112.

[[physics-based-pass-probabilities|Spearman et al.]] reach the same idea by simulated annealing over ball velocities, seven years earlier.

**Nakahara et al. could report this most directly of all** — a Q-value per available action, per player, per timestep, with the observed-versus-maximum gap one subtraction away — and do not. See [[observed-versus-optimal-decisions]] and the prescription task on [[action-valuation-frameworks-compared]].

## The Aggregation Step

Almost every framework ends by summing into a per-90 rating — where [[player-rating-time-series|a season of variation gets discarded]]. [[effective-playing-time|Effective playing time]] varies by team, scoreline and league.

Four frameworks decline the step: [[expected-value-possession-framework|Fernández et al.]] value situations; [[vdep]] and [[gvdep]] aggregate to the team; [[c-obso]] is defined only on shot-ending sequences. Nakahara et al. take a **mean Q-value per player** across evaluated frames rather than a sum, which makes their numbers a *rate* and immune to playing-time confounds — but also unable to reward volume.

## Cross-Sport Lineage

American football — Romer (2006), Yurko et al. (2020). Basketball — [[martingale-epv|Cervone et al.]], Hollinger. Ice hockey — Routley & Schulte (2015), Liu & Schulte (2018). Rugby — Kempton et al. (2016).

The ice-hockey line matters more than its citation count suggests: **it is where the team-as-one-agent RL formulation was established**, and Nakahara et al. define their contribution against it. See [[multi-agent-reinforcement-learning]].

## Common Structural Bias

Van Dijk ranks 81st by VAEP and 142nd by xT.

> ### `offensive-bias-four-causes`
> **Offensive bias has four distinct causes with four different remedies.**
> ^[generated: no source enumerates these, and the fourth was not identifiable before VDEP measured it. Also on [[defensive-valuation]] and the synthesis. rests-on: source:vandijk-rankings, source:mendes-neves-event-data-limits, source:shelopugin-duel-tables, source:vaep-f1-zero]

1. **Definitional** — value is proximity to scoring. → change the target ([[vdep]]).
2. **Data** — event streams cannot judge tackles. → [[optical-tracking-data|tracking]].
3. **Modelling choice** — van Dijk tops both [[duel-skill-rating|duel tables]]. → model those events.
4. **Statistical** — too few positives to train a classifier. → a frequent proxy.

**The fourth is least secure**, because its premise `source:vaep-f1-zero` is under question. If that finding is a thresholding artefact, cause 4 loses its evidence and the decomposition reduces to three.

**Checked 2026-08-07** against Nakahara et al., which is the first framework here to top its ranking with **two centre-backs** (Hatanaka and Thiago, zero season goals between them) — the reverse of the van Dijk result. That is consistent with cause 1: change what value means (from proximity-to-scoring to Q under a movement-inclusive action space) and the positional bias inverts. It is a *demonstration* rather than a test, since no held source computes VAEP and Q on the same players.

**What would falsify the whole.** A single remedy fixing all four at once would show they are facets of one cause. Tracking plausibly addresses 2 and 3 together; nothing addresses 1 and 4 jointly.

## What Remains Invisible

Three gaps have closed: [[off-ball-value|off-ball positioning]], **team** defensive contribution, and [[space-creation|space created for teammates]]. A fourth is computed but unreported — per-defender positioning, see [[drso]].

Still open:

- **Whether the metrics measure the same thing.** Newly the most pressing, see above.
- **Errors of omission** — addressed in principle by per-action Q-values; unreported.
- **Scale** — C-OBSO predicts 3 of 22 players; Nakahara et al. evaluate 14 players from one club, attacking third only.
- **Reliability** — no off-ball metric here reports it.

The general lesson: **a gap in the vault is not a gap in the field**, and inferring the second from the first is how this section came to be wrong twice.

## See Also

- [[expected-possession-value]] · [[expected-threat]] · [[vaep]] · [[vdep]] · [[gvdep]] · [[c-obso]] · [[drso]] · [[obso]] · [[martingale-epv]] · [[pass-carry-reward]]
- [[reinforcement-learning]] · [[multi-agent-reinforcement-learning]] · [[temporal-difference-learning]] · [[action-supervision]] · [[action-space-design]]
- [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[counterfactual-baseline]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[expected-goals]] · [[intent-vs-outcome-valuation]] · [[player-rating-time-series]] · [[model-selection]] · [[theory-based-modelling]] · [[construct-validity]]
- [[temporal-discounting]] · [[possession-risk]] · [[effective-playing-time]] · [[symmetrical-duel-valuation]] · [[duel-skill-rating]]
- [[probability-surface]] · [[policy-modelling]] · [[structured-model-decomposition]] · [[counterfactual-simulation]] · [[pitch-control]]
- [[markov-game]] · [[value-iteration]] · [[action-valuation-frameworks-compared]]
- [[vaep-conceding-classifier]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]] · [[shot-value-formulations-compared]]
- [[physics-based-pass-probabilities|Spearman 2017 Summary]] · [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
