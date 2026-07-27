---
title: "Action Valuation"
type: concept
tags: [sports-analytics, action-valuation, defensive-valuation, player-evaluation, markov-model, evaluation, event-stream-data, time-series, recruitment, discounting, duel-analysis, off-ball, probability-surface, proxy-target, counterfactual]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Action Valuation

Action valuation is the task of assigning a numeric value to each individual action a player performs, reflecting how much that action improved or damaged their team's prospects. It exists because traditional statistics measure almost nothing: shots and assists together account for **less than 1% of all on-the-ball actions** in soccer.

## The Unifying Equation

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] observe that essentially all modern approaches share one form:

$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

An action is worth the change in game-state quality it produces. Every framework differs only in **how it represents $S$** and **how it computes $Q$**.

Deliberately reminiscent of value functions in [[reinforcement-learning]]: $Q$ plays the role of a state-value function and $V(a_i)$ the role of an advantage.

## Three Styles

### Count-based
Weight each action type, then take a weighted sum of counts.
*Examples:* McHale & Scarf (2007); PlayeRank (Pappalardo et al., 2019); Hollinger's PER (2005).
*Limitation:* no regard for context.

### Possession-based
Split the match into possessions, value ball-progressing actions by how they change the chance of a goal *within that possession*. Almost all are [[markov-model|Markov models]] over a discretised state space — the "[[expected-possession-value|expected possession value]]" family.
*Examples:* Rudd (2011); Mackay (2017); [[expected-threat|xT]] (Singh, 2019).
*Limitation:* stops at the turnover, so cannot model risk.

### Action-based
Value a broader set of actions using rich features of action and context, framed as supervised learning.
*Examples:* [[vaep]]; [[pass-carry-reward|Shelopugin]]; [[expected-value-possession-framework|Fernández et al.]]; [[vdep]].
*Limitation:* loses [[interpretability]]; empirically less stable ([[split-half-reliability]]).

## What Distinguishes the Approaches

| Axis | Options | Consequence |
|---|---|---|
| State representation | Zone only → last-$k$ actions → full tracking snapshot | Which actions can be valued at all |
| Horizon | Current possession → next $k$ actions → next goal → unbounded | Whether risk is modelled |
| Estimation | Dynamic programming → supervised learning → Bayesian process model → [[structured-model-decomposition\|decomposed ensemble]] | Interpretability and cost |
| Data | [[event-stream-data]] → [[optical-tracking-data]] | Availability and off-ball coverage |
| **Outcome visibility** | Outcome features included → withheld | Whether execution or *decision* is valued |
| **Credit assignment** | Fixed window → capped decay → hard cutoff → geometric decay | How value propagates back from a goal |
| **Possession attributability** | Assumed → modelled | Whether contested events are visible |
| **Output granularity** | Per action → per location | Whether *unrealised* options can be valued |
| **Perspective** | Attacking → **defending** | Whose success is being measured |
| **Target rarity** | Goals → **frequent proxies** | Whether the classifier can learn at all |

The first four are classical. The rest are recent and underexplored.

**Outcome visibility.** Most frameworks conflate *good decision* with *well executed*. Partitioning features on post-commitment information separates them — see [[intent-vs-outcome-valuation]].

**Credit assignment** now has four positions:

| Approach | Boundary | Framework |
|---|---|---|
| Fixed $k$-action window | Action count ($k=10$) | [[vaep]] |
| Capped time decay | 1 min, floored at 5 actions | [[football-performance-time-series\|Mendes-Neves et al.]] |
| Hard time cutoff | $\epsilon = 15$s, then zero | [[expected-value-possession-framework\|Fernández et al.]] |
| Geometric time decay | None — weight → 0 | [[temporal-discounting\|Shelopugin]] |

Direction of travel: from counting actions to measuring elapsed time. Both time-based approaches carry a free parameter that **neither source subjects to sensitivity analysis** — as does [[vdep]]'s $k=5$ and its $C=3.9$.

## Perspective: Attacking or Defending

Every framework above except [[vdep]] measures **attacking** success and treats defence as its negative — VAEP's $P_{concedes}$, xT's absence of a conceding term, PCR's opponent-side subtraction.

[[football-defence-evaluation-vdep|Toda et al.]] measure this assumption's cost directly. On a 45-match dataset, **VAEP's conceding classifier achieves F1 = 0.000** — it identifies no true positives whatsoever, having learned to predict "no goal" always, which is right 99.2% of the time. The defensive half of the vault's most-cited valuation framework is empirically inert at this data scale.

The fix is to change the target rather than the model. Predicting **ball recovery** and **being attacked** — roughly 90× and 35× more frequent than goals — raises F1 to 0.522 and 0.484. See [[rare-event-proxy-targets]] and [[defensive-valuation]].

Note the metric trap this exposes: VAEP scores *better* on Brier precisely because its target is rarer. Comparing frameworks with different target frequencies on true-negative-sensitive metrics inverts the correct conclusion. See [[class-imbalance-evaluation]].

## The Possession-Attributability Assumption

The unifying equation needs to know *whose* prospects $Q$ describes. Obvious for a pass; undefined for an aerial duel, where two opposing players contest a ball neither holds.

Every framework except Shelopugin's resolves this by exclusion. [[symmetrical-duel-valuation]] closes it, and exposes the still-unsolved problem of **credit-splitting between passer and receiver**.

## Valuing What Happened vs What Was Available

Every framework except [[expected-value-possession-framework|Fernández et al.]] values actions that occurred. A full [[probability-surface|surface]] over pass destinations changes the question to "how good was that pass *relative to what was available*?"

Realised EPV 0.032 against best-available 0.112 in their worked example; **the gap is the coaching output**. Structurally the RL advantage function, repurposed as an interpretive device — see [[policy-modelling]].

The same shift is what makes individual defensive credit tractable, by asking where a defender *could* have stood. See below.

## The Aggregation Step

Almost every framework ends by summing action values into a per-90 rating — where [[player-rating-time-series|a season of variation gets discarded]].

The denominator deserves equal scrutiny: [[effective-playing-time|effective playing time]] varies by team, scoreline and league, so per-90 favours players at high-tempo sides.

Two frameworks decline the step entirely, for different reasons. [[expected-value-possession-framework|Fernández et al.]] value situations, not seasons. [[vdep]] aggregates to the **team**, because defensive credit cannot be individuated *by that method*. Both are consequently immune to the reliability critique and unusable for [[recruitment]].

## Cross-Sport Lineage

- American football — Romer (2006); Yurko et al. (2020)
- Basketball — [[martingale-epv|Cervone et al.]] (2014, 2016); Hollinger (2005)
- Ice hockey — Routley & Schulte (2015); Liu & Schulte (2018)
- Rugby — Kempton et al. (2016)

## Common Structural Bias

Every attacking-perspective framework rewards offensive actions more richly, because value is defined via proximity to scoring. Van Dijk ranks 81st by VAEP and 142nd by xT.

**Four causes, four remedies** — separating them matters, because they are not fixed by the same thing:

1. **Definitional.** Value is proximity to scoring. Remedy: change the target ([[vdep]]).
2. **Data.** [[event-stream-data|Event data]] cannot judge tackles and interceptions well. Remedy: [[optical-tracking-data|tracking]].
3. **Modelling choice.** Van Dijk tops both of [[duel-skill-rating|Shelopugin's duel tables]] — the information exists in ordinary event data, unmodelled. Remedy: model those events.
4. **Statistical.** The conceding classifier cannot learn from 227 positives. Remedy: a frequent proxy.

The fourth is new and is the one nobody had measured.

## What Remains Invisible

Two gaps have closed partially. [[off-ball-value|Off-ball positioning]] is now readable from pass surfaces, and [[vdep]] makes **team** defensive contribution measurable from all 22 positions.

**Correction, 2026-07-27.** This section previously listed individual defensive credit as invisible to every framework. That is true of the frameworks *held in this vault*, and false of the literature. **Umemoto & Fujii (2023)** evaluate individual defenders by counterfactual positioning — searching which grid cell a defender could have occupied to most reduce the opponent's off-ball scoring opportunity. **Teranishi, Tsutsui, Takeda & Fujii (2022/23)** credit movement sacrificed to create space for a teammate, via trajectory prediction. Neither is held in `raw/`; both are cited only, and their capability claims are unverified here. See [[defensive-valuation]].

Still invisible to every framework **read here**:

- **Individual defensive credit** — VDEP is explicitly team-level, and no held framework individuates it.
- **Credit for creating space someone else exploits** — the run that drags a marker rewards the beneficiary, not the runner.
- **Movement over time** — arriving in a good position by a clever run scores the same as standing there.
- **Errors of omission** — a defender who fails to press generates no event.

The general lesson is worth keeping separate from the specifics: **a gap in the vault is not a gap in the field**, and inferring the second from the first is how this section came to be wrong.

## See Also

- [[expected-possession-value]] · [[expected-threat]] · [[vaep]] · [[vdep]] · [[martingale-epv]] · [[pass-carry-reward]]
- [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[expected-goals]] · [[intent-vs-outcome-valuation]] · [[player-rating-time-series]]
- [[temporal-discounting]] · [[possession-risk]] · [[effective-playing-time]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]] · [[off-ball-value]]
- [[probability-surface]] · [[policy-modelling]] · [[structured-model-decomposition]] · [[counterfactual-simulation]]
- [[markov-game]] · [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
- [[epv-control-duel-skills-football|EPV Control and Duel Summary]] · [[expected-value-possession-framework|Soccer EPV Framework Summary]]
- [[football-defence-evaluation-vdep|VDEP Summary]]
