---
title: "Expected Possession Value (EPV)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, markov-model, evaluation, statistics, time-series, discounting, duel-analysis, off-ball, probability-surface, model-decomposition]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md, raw/papers/football-performance-time-series.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.95
provenance:
  extracted: 70%
  inferred: 27%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Expected Possession Value (EPV)

Expected possession value is the idea that a possession in progress has a *current worth* — the expected points (or goals) it will eventually yield, given everything that has happened so far. Actions are then valued by how much they move that number.

$$\text{EPV at time } t = \mathbb{E}[\text{eventual outcome} \mid \text{everything observed so far}]$$

The appeal is intuitive: it matches how coaches and spectators already think. A possession feels more or less promising as it unfolds, and a good action is one that makes it more promising.

## ⚠️ The Term Means Three Different Things

The main source of confusion in the literature, and the reason this page exists separately from the model pages.

### 1. In basketball: a specific model
[[martingale-epv|Cervone et al.'s (2016) construction]] — a continuous-time [[martingale]] estimated from 25 Hz [[optical-tracking-data]] via a [[multiresolution-modelling|multiresolution]] stochastic process model. "EPV" here names one particular, computationally enormous piece of work.

### 2. In soccer: a category of models
[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] use "expected possession value (EPV) approaches" as a **taxonomic label** for one of three styles of [[action-valuation]]: possession-based Markov models that divide a match into possessions and value ball-progressing actions — [[expected-threat|xT]] (Singh, 2019), Rudd (2011), Mackay (2017), Yam (2019). These are typically a transition matrix over pitch zones solved by [[value-iteration]].

### 3. In soccer: a specific tracking-data framework
[[expected-value-possession-framework|Fernández, Bornn & Cervone]] — a genuine Cervone-style construction brought *to* soccer, and the paper most people mean when they say "soccer EPV" in a technical register. Nine neural components, tracking data, real-time.

[[epv-control-duel-skills-football|Shelopugin]] adds a fourth usage: a supervised **event-data** model whose target is accumulated future [[expected-goals|xG]] — neither the basketball martingale nor a zonal Markov chain, closer in machinery to [[vaep]] while retaining the possession framing.

**These are not interchangeable.** The category label (2) includes models that would fail the criteria of both (1) and (3).

> **Dating note, resolved.** The Fernández–Bornn–Cervone framework exists in three versions: a 2019 MIT Sloan conference paper, the 2020 arXiv preprint (arXiv:2011.09426, held in `raw/`), and the 2021 *Machine Learning* 110(6), 1389–1427 journal article. All three are the same work at increasing length. Earlier vault notes flagged uncertainty between 2019 and 2021; both are defensible depending on version.

## The Shared Ideology

Despite the naming mess, everything under this banner rests on the same three commitments.

### 1. Possession as the unit of value
Value accrues within a possession and resets at a turnover. This distinguishes EPV-style thinking from shot-centric metrics like [[expected-goals|xG]] — the entire build-up matters, not just the finish.

Note that the strongest members of the family **abandon this commitment** in its literal form. Fernández et al. define possession as ending at the *next goal*, not the next turnover; Shelopugin lets decayed risk run over unboundedly many possessions. The unit survives conceptually while the boundary dissolves.

### 2. Actions valued by the change they cause
$$V(a_i) = Q(S_i) - Q(S_{i-1})$$

The general [[action-valuation]] equation. EPV approaches supply $Q$ by asking "how many points/goals is this position worth?"

### 3. Value defined by lookahead, not outcome
The value of a position is the expected *future* payoff, not what actually happened next. Every implementation needs a mechanism to propagate value backwards from scoring events — [[value-iteration]] in xT, transition-matrix algebra in the basketball model, supervised lookahead labelling in [[vaep]], Shelopugin, and Fernández et al.

## What the Target Actually Is

Easy to gloss over, and the frameworks differ more here than anywhere else:

| Framework | Target | Signed? |
|---|---|---|
| [[expected-threat\|xT]] | P(goal this possession) | No |
| [[vaep]] | P(score) − P(concede) over next 10 actions | Yes, by construction |
| [[pass-carry-reward\|Shelopugin]] | Decayed future xG, team minus opponent | Yes |
| **Fernández et al.** | $G \in \{-1, 1\}$ — **which team scores next** | Yes, natively |

The last is the cleanest. Because the target *is* which team scores next, conceding is not a separately-modelled penalty but the negative half of the same variable. [[possession-risk|Risk]] is intrinsic rather than bolted on.

## Where Implementations Diverge

| Design choice | Options | Consequence |
|---|---|---|
| **Horizon** | Current possession → next $k$ actions → next goal → unbounded | Determines whether *risk* can be modelled at all |
| **State** | Ball zone → last-$k$ actions → full tracking snapshot | Determines which actions are visible |
| **Estimation** | Dynamic programming vs supervised learning vs Bayesian process model vs [[structured-model-decomposition\|decomposed ensemble]] | Determines interpretability, cost, martingale property |
| **Data** | [[event-stream-data\|Event stream]] vs [[optical-tracking-data\|tracking]] | Determines availability and off-ball coverage |
| **Credit decay** | Hard action window → capped time decay → **hard time cutoff** → geometric decay | How sharply value attribution cuts off before a goal |
| **Time base** | Clock minutes vs [[effective-playing-time\|effective playing time]] | Whether dead time dilutes credit |
| **Contested events** | Ignored vs valued separately | Whether duels are visible at all |
| **Output granularity** | Single value → value per action → **[[probability-surface\|value per location]]** | Whether unrealised options can be valued |

The horizon choice is the sharpest. A strictly possession-bounded model (xT) *cannot* value how an action changes the chance of conceding. [[vaep]] breaks the boundary with a $k=10$ action window; Fernández et al. extend all the way to the next goal; Shelopugin removes the boundary entirely via decay.

The credit-decay row now has four positions. Fernández et al. contribute a **hard temporal cutoff**: rewards vanish beyond $\epsilon = 15$s, chosen as the mean possession duration. That sits between VAEP's action-count window and Shelopugin's continuous decay — temporal like the latter, discontinuous like the former. Neither $\epsilon$ nor $\gamma$ is supported by sensitivity analysis in its source.

The last row is new and arguably the most consequential. See below.

## From Valuing Actions to Valuing Options

Every framework except Fernández et al. values **what happened**. Estimating a full [[probability-surface]] over pass destinations changes the question to what was *available*.

Two things follow that no other framework here provides:

- **The gap between realised and optimal.** In the source's worked example, realised EPV is 0.032 while the best available pass would have yielded 0.112. That gap — not either number — is the coaching output. See [[policy-modelling]].
- **[[off-ball-value|Off-ball value]] for free.** If every location has a pass value, the worth of a player *standing* there is immediately available, without modelling off-ball behaviour at all.

## Blind Spots Closed and Still Open

Two structural gaps have now been addressed by different sources, from different data:

| Gap | Closed by | How |
|---|---|---|
| Contested events (duels) | [[symmetrical-duel-valuation\|Shelopugin]] | Event data; duel inherits the following control action's value |
| Off-ball positioning | [[off-ball-value\|Fernández et al.]] | Tracking data; pass surface read at player locations |

Neither closes the other's, and no framework does both. Still open across all of them: **defensive off-ball work** — pressing, screening, marking — which suppresses the opponent's value rather than generating one's own, and which nothing here attributes to individuals.

## Use Cases

- **Player rating** — sum action values per 90 (or per 60 effective minutes; see [[pass-carry-reward|PCR]]). The dominant application, and where frameworks diverge most.
- **Playing style characterisation** — decompose value by action type; over time this reveals style change, see [[player-rating-time-series]].
- **Scouting** — identify players creating value invisible to goals and assists.
- **Transfer forecasting** — project the metric forward under a change of club; see [[transfer-performance-prediction]].
- **[[tactical-analysis|Tactical and opposition analysis]]** — locate where a team gains or concedes value, conditioned on opponent shape.
- **Live decision support** — Fernández et al.'s control room, running at 89× the tracking frame rate.

## Common Limitations

1. **Offensive bias.** Value is defined by proximity to scoring, so defenders are systematically undervalued. Van Dijk ranks 81st by VAEP and 142nd by xT — though he tops both of Shelopugin's duel-rating tables, suggesting this is partly a choice about which events get modelled.
2. **On-ball only** — now with the partial exception above.
3. **No ground truth.** When frameworks disagree about an action's value, nothing adjudicates.
4. **Context dependence.** Accumulating value is easier in a weaker league or stronger team.
5. **No cross-framework benchmark.** Every source reports losses against its own baseline or none at all. Nothing has ever been compared to anything on a shared task.

## Implementations in This Vault

- [[martingale-epv]] — basketball, tracking, martingale-guaranteed, 461 processors
- [[expected-value-possession-framework|Fernández et al.]] — soccer, tracking, decomposed, surfaces, off-ball, real-time
- [[expected-threat]] — soccer, zonal, interpretable, highly reliable, offence-only
- [[vaep]] — soccer, event data, contextual, risk-aware, low reliability
- [[pass-carry-reward|Shelopugin / PCR]] — soccer, event data, time-decayed, duel-aware, reliability unreported
- [[expected-goals]] — shots only; a *component* of the others rather than a competitor

## See Also

- [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[martingale-epv]] · [[expected-threat]] · [[vaep]] · [[pass-carry-reward]]
- [[probability-surface]] · [[soccermap]] · [[off-ball-value]] · [[pitch-control]]
- [[structured-model-decomposition]] · [[policy-modelling]] · [[probability-calibration]]
- [[possession-risk]] · [[temporal-discounting]] · [[effective-playing-time]] · [[symmetrical-duel-valuation]]
- [[javier-fernandez]] · [[luke-bornn]] · [[daniel-cervone]]
