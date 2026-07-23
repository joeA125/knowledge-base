---
title: "Expected Possession Value (EPV)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, markov-model, evaluation, statistics]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.9
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
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

**The two usages are not interchangeable.** The soccer category is broad and includes models that would fail the basketball model's own stated criteria (notably stochastic consistency). Fernández, Bornn & Cervone (2019) complicate matters further by bringing a genuine Cervone-style framework *to* soccer using tracking data.

## The Shared Ideology

Despite the naming mess, everything under this banner rests on the same three commitments.

### 1. Possession as the unit of value
Value accrues within a possession and resets at a turnover. This is what distinguishes EPV-style thinking from shot-centric metrics like [[expected-goals|xG]] — the entire build-up matters, not just the finish.

### 2. Actions valued by the change they cause
$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

The general [[action-valuation]] equation. EPV approaches supply $Q$ by asking "how many points/goals is this position worth?"

### 3. Value defined by lookahead, not outcome
The value of a position is the expected *future* payoff from it, not what actually happened next. This is why every implementation needs some mechanism to propagate value backwards from scoring events — [[value-iteration]] in xT, transition-matrix algebra in the basketball model, supervised lookahead labelling in [[vaep]].

## Where Implementations Diverge

| Design choice | Options | Consequence |
|---|---|---|
| **Horizon** | Current possession only vs beyond turnovers | Determines whether *risk* (conceding) can be modelled at all |
| **State** | Ball zone → last-$k$ actions → full tracking history | Determines which actions are visible |
| **Estimation** | Dynamic programming vs supervised learning vs Bayesian process model | Determines interpretability, cost, and whether the martingale property holds |
| **Data** | [[event-stream-data\|Event stream]] vs [[optical-tracking-data\|tracking]] | Determines availability and off-ball coverage |

The horizon choice is the sharpest. A strictly possession-bounded model (xT) *cannot* value how an action changes the chance of conceding, because conceding happens after the possession ends. [[vaep]] deliberately breaks the possession boundary — looking $k=10$ actions ahead regardless of turnovers — precisely to capture that risk. This is why VAEP is classified as *action-based* rather than *possession-based* despite sharing the same underlying equation.

## Use Cases

- **Player rating** — sum action values per 90 minutes. The dominant application, and where the frameworks diverge most in their conclusions.
- **Playing style characterisation** — decompose value by action type to distinguish a dribbler from a passer.
- **Scouting** — identify players creating value invisible to goals and assists.
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
- [[expected-goals]] — shots only; a *component* of the others rather than a competitor

## See Also

- [[action-valuation]]
- [[action-valuation-frameworks-compared]]
- [[martingale-epv]]
- [[expected-threat]]
- [[vaep]]
- [[markov-game]]
- [[value-iteration]]
