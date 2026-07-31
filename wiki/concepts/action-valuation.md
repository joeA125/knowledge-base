---
title: "Action Valuation"
type: concept
tags: [sports-analytics, action-valuation, defensive-valuation, off-ball, space-creation, player-evaluation, markov-model, evaluation, event-stream-data, time-series, recruitment, discounting, duel-analysis, probability-surface, proxy-target, counterfactual]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.9
provenance:
  extracted: 70%
  inferred: 18%
  generated: 8%
  imported: 0%
  ambiguous: 4%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Action Valuation

Assigning a numeric value to each action a player performs, reflecting how much it improved or damaged their team's prospects. It exists because traditional statistics measure almost nothing: shots and assists together are **less than 1% of all on-the-ball actions**.

## The Unifying Equation

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]]: essentially all modern approaches share one form.

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

Frameworks differ only in **how they represent $S$** and **how they compute $Q$**. Deliberately reminiscent of [[reinforcement-learning]]: $Q$ is a state-value function, $V(a_i)$ an advantage — though almost none of this work is reinforcement learning, since nobody is learning a policy by interaction.

## Four Styles

**Count-based** — weight each action type, sum counts. McHale & Scarf (2007); PlayeRank (2019). *No regard for context.*

**Possession-based** — value ball-progressing actions by how they change the goal chance *within a possession*. Mostly [[markov-game|Markov models]] solved by [[value-iteration]]. [[expected-threat|xT]] (2019). *Stops at the turnover, so cannot model risk.*

**Action-based** — rich features of action and context, framed as supervised learning. [[vaep]]; [[pass-carry-reward|Shelopugin]]; [[expected-value-possession-framework|Fernández et al.]]; [[vdep]]. *Loses [[interpretability]]; less stable.*

**Counterfactual** — value an action or position by comparison against a predicted reference rather than a learned value function. [[c-obso]] and Umemoto & Fujii's defensive positioning.^[generated: this fourth style is added here; Van Roy et al.'s taxonomy has three, and C-OBSO fits none of them] See [[counterfactual-baseline]].

## What Distinguishes the Approaches

| Axis | Options | Consequence |
|---|---|---|
| State representation | Zone → last-$k$ actions → tracking snapshot | Which actions can be valued |
| Horizon | Possession → next $k$ actions → next goal → unbounded | Whether risk is modelled |
| Estimation | DP → supervised → Bayesian process → [[structured-model-decomposition\|decomposed]] → counterfactual | Interpretability and cost |
| Data | [[event-stream-data]] → [[optical-tracking-data]] | Availability and off-ball coverage |
| **Outcome visibility** | Included → withheld | Execution or *decision* |
| **Credit assignment** | Fixed window → capped decay → hard cutoff → geometric decay | How value propagates back |
| **Possession attributability** | Assumed → modelled | Whether duels are visible |
| **Output granularity** | Per action → per location | Whether *unrealised* options can be valued |
| **Perspective** | Attacking → defending | Whose success is measured |
| **Target rarity** | Goals → frequent proxies | Whether the classifier can learn |
| **Whose value** | The actor → **a teammate's** | Whether space creation is credited |

**Credit assignment** now has four positions ([[vaep]]'s $k=10$; a capped time decay; Fernández et al.'s $\epsilon = 15$s; [[temporal-discounting|Shelopugin's]] geometric decay). Every time-based approach carries a free parameter that **no source subjects to sensitivity analysis** — as does [[vdep]]'s $k=5$ and $C=3.9$, and [[c-obso]]'s 4-second window. See [[free-parameters-load-bearing]].

## Perspective: Attacking or Defending

Every framework above except [[vdep]] measures **attacking** success and treats defence as its negative.

[[football-defence-evaluation-vdep|Toda et al.]] report VAEP's conceding classifier at **F1 = 0.000** on a 45-match dataset. ⚠️ That figure needs care — it is near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. See [[vaep-conceding-classifier]].

The proposed fix is to change the target rather than the model: predicting **ball recovery** and **being attacked** — roughly 90× and 35× more frequent — raises F1 to 0.522 and 0.484. See [[rare-event-proxy-targets]].

## Whose Value: Actor or Beneficiary

Every framework above except [[c-obso]] credits the player **performing** the valued act. C-OBSO credits the player whose movement improved *someone else's* chance.

This is relational credit, and no other framework here expresses it. On the same 15 players, C-OBSO correlates 0.45 with annual salary while OBSO (−0.28) and goals (−0.23) do not. See [[space-creation]].

## The Possession-Attributability Assumption

The equation needs to know *whose* prospects $Q$ describes. Obvious for a pass, undefined for an aerial duel. Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it, and exposes the still-unsolved credit-splitting between passer and receiver.

## Valuing What Happened vs What Was Available

A full [[probability-surface|surface]] over pass destinations changes the question to "how good was that pass *relative to what was available*?" Realised 0.032 against best-available 0.112; **the gap is the coaching output**. See [[policy-modelling]] and [[observed-versus-optimal-decisions]].

## The Aggregation Step

Almost every framework ends by summing into a per-90 rating — where [[player-rating-time-series|a season of variation gets discarded]]. The denominator deserves equal scrutiny: [[effective-playing-time|effective playing time]] varies by team, scoreline and league.

Three frameworks decline the step. [[expected-value-possession-framework|Fernández et al.]] value situations. [[vdep]] aggregates to the team, because defensive credit cannot be individuated *by that method*. [[c-obso]] is defined only on shot-ending sequences.

## Cross-Sport Lineage

American football — Romer (2006), Yurko et al. (2020). Basketball — [[martingale-epv|Cervone et al.]], Hollinger. Ice hockey — Routley & Schulte (2015). Rugby — Kempton et al. (2016).

## Common Structural Bias

Every attacking-perspective framework rewards offensive actions more richly. Van Dijk ranks 81st by VAEP and 142nd by xT.

> ^[generated: this four-way decomposition is constructed here. No source enumerates these causes, and the fourth rests on the F1 evidence flagged above. It also appears on [[defensive-valuation]] and the synthesis.]

**Four causes, four remedies:**

1. **Definitional** — value is proximity to scoring. → change the target ([[vdep]]).
2. **Data** — event streams cannot judge tackles. → [[optical-tracking-data|tracking]].
3. **Modelling choice** — van Dijk tops both [[duel-skill-rating|duel tables]]; the information exists unmodelled. → model those events.
4. **Statistical** — too few positives to train a classifier. → a frequent proxy.

The fourth is the least secure, since the evidence for it is the F1 figure now under question.

## What Remains Invisible

Three gaps have closed. [[off-ball-value|Off-ball positioning]] is readable from pass surfaces; [[vdep]] makes **team** defensive contribution measurable; [[c-obso]] credits [[space-creation|space created for teammates]].

Still open:

- **Individual defensive credit** — addressed in the literature (Umemoto & Fujii, 2023) but not held here.
- **Movement over time beyond short windows** — C-OBSO uses 4 s.
- **Errors of omission** — a defender who fails to press generates no event.
- **Scale** — C-OBSO predicts 3 of 22 players.

The general lesson: **a gap in the vault is not a gap in the field**, and inferring the second from the first is how this section came to be wrong twice.

## See Also

- [[expected-possession-value]] · [[expected-threat]] · [[vaep]] · [[vdep]] · [[c-obso]] · [[obso]] · [[martingale-epv]] · [[pass-carry-reward]]
- [[defensive-valuation]] · [[off-ball-value]] · [[space-creation]] · [[counterfactual-baseline]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[expected-goals]] · [[intent-vs-outcome-valuation]] · [[player-rating-time-series]] · [[model-selection]]
- [[temporal-discounting]] · [[possession-risk]] · [[effective-playing-time]] · [[symmetrical-duel-valuation]] · [[duel-skill-rating]]
- [[probability-surface]] · [[policy-modelling]] · [[structured-model-decomposition]] · [[counterfactual-simulation]]
- [[markov-game]] · [[reinforcement-learning]] · [[action-valuation-frameworks-compared]]
- [[vaep-conceding-classifier]] · [[free-parameters-load-bearing]] · [[observed-versus-optimal-decisions]]
- [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
