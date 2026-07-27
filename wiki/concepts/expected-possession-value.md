---
title: "Expected Possession Value (EPV)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, markov-model, evaluation, statistics, time-series, discounting, duel-analysis]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Expected Possession Value (EPV)

Expected possession value is the idea that a possession in progress has a *current worth* — the expected points (or goals) it will eventually yield, given everything that has happened so far. Actions are then valued by how much they move that number.

$$\text{EPV at time } t = \mathbb{E}[\text{eventual outcome} \mid \text{everything observed so far}]$$

The appeal is intuitive: it matches how coaches and spectators already think. A possession feels more or less promising as it unfolds, and a good action is one that makes it more promising.

## ⚠️ The Term Means Two Different Things

This is the main source of confusion in the literature, and the reason this page exists separately from the model pages.

### In basketball: a specific model
[[martingale-epv|Cervone et al.'s (2016) construction]] — a continuous-time [[martingale]] estimated from 25 Hz [[optical-tracking-data]] via a [[multiresolution-modelling|multiresolution]] stochastic process model. "EPV" here names one particular, computationally enormous piece of work.

### In soccer: a category of models
[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] use "expected possession value (EPV) approaches" as a **taxonomic label** for one of three styles of [[action-valuation]]: possession-based Markov models that divide a match into possessions and value ball-progressing actions. The category includes:

- [[expected-threat|xT]] (Singh, 2019)
- Rudd (2011) — Markov chains for tactical analysis
- Mackay (2017) — "xG added"
- Yam (2019) — attacking contributions

These are far simpler than the basketball model — typically a transition matrix over pitch zones solved by [[value-iteration]].

**The two usages are not interchangeable.** The soccer category is broad and includes models that would fail the basketball model's own stated criteria (notably stochastic consistency). [[javier-fernandez|Fernández]], Bornn & Cervone complicate matters further by bringing a genuine Cervone-style framework *to* soccer using tracking data.

A third usage is now visible in the vault: [[epv-control-duel-skills-football|Shelopugin]] uses "EPV" for a supervised **event-data** model whose target is accumulated future [[expected-goals|xG]]. This is neither the basketball martingale nor a zonal Markov chain — it is closer in machinery to [[vaep]] while retaining the possession framing. The term is now stretched across three quite different objects.

> **Citation note.** This vault previously dated the Fernández–Bornn–Cervone soccer EPV framework to 2019. The journal version is **2021** — *Machine Learning* 110(6), 1389–1427 (June 2021), "A framework for the fine-grained evaluation of the instantaneous expected value of soccer possessions" — as cited by [[football-performance-time-series|Mendes-Neves et al.]] A 2019 conference version (MIT Sloan) also exists, so both dates appear in the literature; the 2021 journal reference is the fuller one, and Shelopugin cites the 2019 one.

## The Shared Ideology

Despite the naming mess, everything under this banner rests on the same three commitments.

### 1. Possession as the unit of value
Value accrues within a possession and resets at a turnover. This is what distinguishes EPV-style thinking from shot-centric metrics like [[expected-goals|xG]] — the entire build-up matters, not just the finish.

### 2. Actions valued by the change they cause
$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

The general [[action-valuation]] equation. EPV approaches supply $Q$ by asking "how many points/goals is this position worth?"

### 3. Value defined by lookahead, not outcome
The value of a position is the expected *future* payoff from it, not what actually happened next. This is why every implementation needs some mechanism to propagate value backwards from scoring events — [[value-iteration]] in xT, transition-matrix algebra in the basketball model, supervised lookahead labelling in [[vaep]] and in Shelopugin's model.

## Where Implementations Diverge

| Design choice | Options | Consequence |
|---|---|---|
| **Horizon** | Current possession only vs beyond turnovers vs unbounded | Determines whether *risk* (conceding) can be modelled at all |
| **State** | Ball zone → last-$k$ actions → full tracking history | Determines which actions are visible |
| **Estimation** | Dynamic programming vs supervised learning vs Bayesian process model | Determines interpretability, cost, and whether the martingale property holds |
| **Data** | [[event-stream-data\|Event stream]] vs [[optical-tracking-data\|tracking]] | Determines availability and off-ball coverage |
| **Credit decay** | Hard action-count window vs capped time decay vs geometric decay | Determines how sharply value attribution cuts off before a goal |
| **Time base** | Clock minutes vs [[effective-playing-time\|effective playing time]] | Determines whether dead time dilutes credit and normalisation |
| **Contested events** | Ignored vs valued separately | Determines whether duels are visible to the model at all |

The horizon choice is the sharpest. A strictly possession-bounded model (xT) *cannot* value how an action changes the chance of conceding, because conceding happens after the possession ends. [[vaep]] deliberately breaks the possession boundary — looking $k=10$ actions ahead regardless of turnovers — precisely to capture that risk. This is why VAEP is classified as *action-based* rather than *possession-based* despite sharing the same underlying equation.

The credit-decay row is a subtler variant of the same question. A hard $k$-action window makes the $k$-th previous action fully in scope and the $(k{+}1)$-th fully out; [[football-performance-time-series|Mendes-Neves et al.]] replace this with a time-decayed label, capped at one minute and floored for the last five actions, making the boundary continuous. [[temporal-discounting|Shelopugin's geometric decay]] removes the cap entirely — and in doing so removes the need for a horizon at all, since weight simply falls to zero. See [[possession-risk]].

The last two rows are Shelopugin's additions and are unaddressed elsewhere in the vault. See [[effective-playing-time]] and [[symmetrical-duel-valuation]].

## The Contested-Event Blind Spot

Every implementation above except Shelopugin's assumes **possession is attributable** at each event. That assumption silently excludes aerial and ground duels, where two opposing players contest a ball neither holds.

This is not a minor gap. Duels are where physical mismatch is decisive, and a valuation framework blind to them cannot distinguish a long ball to a dominant target from the same ball to a weak one. [[symmetrical-duel-valuation]] sets out the mechanism for closing it, and the [[epv-control-duel-skills-football|Donnarumma case study]] quantifies what is lost: a duel-blind model values two tactically identical passes at 0.00075 and 0.00077, while a duel-aware model separates them by roughly a factor of two.

## Use Cases

- **Player rating** — sum action values per 90 minutes (or per 60 effective minutes; see [[pass-carry-reward|PCR]]). The dominant application, and where the frameworks diverge most in their conclusions.
- **Playing style characterisation** — decompose value by action type to distinguish a dribbler from a passer. Tracked *over time*, this reveals style change — see [[player-rating-time-series]].
- **Scouting** — identify players creating value invisible to goals and assists.
- **Transfer forecasting** — project the resulting metric forward under a change of club; see [[transfer-performance-prediction]].
- **Tactical analysis** — locate where on the pitch a team gains or loses value.
- **Live broadcast** — the basketball model's stock-ticker curve is designed for exactly this.

## Common Limitations

Shared across every implementation:

1. **Offensive bias.** Value is defined by proximity to scoring, so defenders are systematically undervalued. Virgil van Dijk ranks 81st by VAEP and 142nd by xT.
2. **On-ball only** (except tracking-based models, partially). Pressing, marking and off-ball movement are invisible.
3. **No ground truth.** When two frameworks disagree about an action's value, there is no way to adjudicate.
4. **Context dependence.** Accumulating value is easier in a weaker league or a stronger team.

## Implementations in This Vault

- [[martingale-epv]] — basketball, tracking data, martingale-guaranteed, extremely expensive
- [[expected-threat]] — soccer, zonal, interpretable, highly reliable, offence-only
- [[vaep]] — soccer, contextual, risk-aware, low reliability
- [[pass-carry-reward|Shelopugin's EPV / PCR]] — soccer, event data, time-decayed, duel-aware, reliability unreported
- [[expected-goals]] — shots only; a *component* of the others rather than a competitor

## See Also

- [[action-valuation]]
- [[action-valuation-frameworks-compared]]
- [[martingale-epv]] · [[expected-threat]] · [[vaep]] · [[pass-carry-reward]]
- [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]]
- [[symmetrical-duel-valuation]] · [[duel-skill-rating]]
- [[intent-vs-outcome-valuation]] · [[player-rating-time-series]]
- [[markov-game]] · [[value-iteration]]
- [[javier-fernandez]]
