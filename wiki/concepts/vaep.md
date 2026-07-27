---
title: "VAEP (Valuing Actions by Estimating Probabilities)"
type: concept
tags: [machine-learning, sports-analytics, statistics, evaluation, player-evaluation, probabilistic-classification, markov-model, action-valuation, defensive-valuation, reliability, regression, random-forest, time-series, class-imbalance]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/football-performance-time-series.md, raw/papers/football_defence_evaluation.md]
confidence: 0.95
provenance:
  extracted: 85%
  inferred: 12%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
---

# VAEP (Valuing Actions by Estimating Probabilities)

VAEP ([[evaluating-football-player-actions|Decroos et al., 2019]]) is a framework for assigning a value to every on-ball action in a soccer game based on its impact on the probability of scoring and conceding in the near future. It is the canonical *action-based* approach in the [[action-valuation]] taxonomy.

## Core Idea

For each game state $S_i$, estimate the probability of the team in possession scoring ($P_{scores}$) and conceding ($P_{concedes}$) within the next $k=10$ actions. An action's value is the change it causes:

$$V(a_i) = \underbrace{\Delta P_{scores}(a_i)}_{\text{offensive value}} + \underbrace{(-\Delta P_{concedes}(a_i))}_{\text{defensive value}}$$

## Why VAEP Improves on Traditional Metrics

| Limitation of traditional metrics | How VAEP addresses it |
|---|---|
| Only count goals and assists | Values all 21 [[spadl]] action types |
| Fixed value regardless of context | Context-aware: location, game state, preceding actions |
| Only immediate effects | Considers effects up to 10 actions ahead |

This is why VAEP identifies players like De Bruyne and Hazard who are invisible to goals/assists metrics.

## Why It Is Not "Possession-Based"

VAEP deliberately breaks the possession boundary. Where the [[expected-possession-value|possession-based family]] estimates the chance of a goal *within the current possession*, VAEP looks $k=10$ actions ahead regardless of turnovers. That choice is what lets it model **risk**, and is why it is classified as action-based rather than as an EPV approach.

## ⚠️ The Defensive Half Does Not Work

The most consequential finding about VAEP in this vault, and it comes from outside the VAEP line of work.

[[football-defence-evaluation-vdep|Toda et al. (2022)]] re-implement VAEP on 45 J-League matches and evaluate its two classifiers with the **F1 score** alongside the usual AUC and Brier:

| Classifier | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | 0.003 | **0.000** |

**$P_{concedes}$ identifies no true positives at all.** It has learned to predict "no goal conceded" always — correct 99.2% of the time on 227 positives across 97,335 events, and completely uninformative. The defensive half of $V(a_i)$ is therefore not weak but *empirically inert* at this data scale.

This resolves something the vault had recorded without explanation: VAEP correlates essentially **zero with goals conceded** ($r = -0.098$ across a season, $-0.040$ within a match), despite being explicitly constructed from a conceding model. If $\Delta P_{concedes}$ is near-constant, $V(a_i)$ reduces in practice to its offensive term.

**Why the standard metrics hid this.** $P_{concedes}$ has the *best Brier score in the comparison* (0.003) precisely because its target is rarest — squared error against a near-zero base rate is tiny. Brier, AUC and accuracy are all inflated by the enormous true-negative mass. Only F1, which ignores true negatives entirely, exposes the failure. See [[class-imbalance-evaluation]].

**How much this generalises is genuinely open.** Decroos et al. trained on a far larger corpus (8.5M actions), where the absolute number of conceding events is much greater. The honest statement is that VAEP's conceding classifier fails **at the data scale most researchers and clubs actually have**, and that nobody has reported its F1 on the original corpus. That measurement would be cheap and valuable.

The constructive response is not a better classifier but a different target — predicting frequent events on the causal path to conceding rather than conceding itself. See [[rare-event-proxy-targets]], [[vdep]] and [[defensive-valuation]].

## Data Foundation

VAEP is built on [[spadl]] representations of [[event-stream-data]]. The fixed 9-attribute action format makes it possible to construct the fixed-length feature vectors the probability estimators require.

## Probability Estimation

Two [[probabilistic-classification|probabilistic classifiers]] ([[gradient-boosting|CatBoost]]; XGBoost in later work) estimate $P_{scores}$ and $P_{concedes}$ from features of the previous 3 actions. CatBoost outperforms Logistic Regression, [[random-forest|Random Forest]], and XGBoost on both [[probability-calibration|Brier score and ROC AUC]].

Because action values are computed by **summing and subtracting** probabilities, [[probability-calibration|calibration]] is essential. Note the finding above, though: calibration is necessary and *not sufficient* — a classifier predicting the base rate is perfectly calibrated and finds nothing.

## Variants: I-VAEP and O-VAEP

[[football-performance-time-series|Mendes-Neves, Meireles & Mendes-Moreira]] build a modified VAEP departing from the original in three ways:

| | Original (Decroos et al.) | Mendes-Neves et al. |
|---|---|---|
| Target | Two probabilities | One continuous label on $[-1, 1]$ |
| Framing | [[probabilistic-classification\|Classification]] | Regression |
| Label horizon | Next $k = 10$ actions | Time-decayed (capped at 1 min), floored at last 5 actions |
| Learner | [[gradient-boosting\|CatBoost]] | [[random-forest\|Random Forest]] |
| Differencing | Consecutive states | **Lag of two actions** |

The lag-of-two choice addresses paired actions — a foul followed by a dribble should contribute jointly.

Collapsing two probabilities into one signed target means offensive and defensive value are no longer separately estimated. This loses independent inspection of each, but sidesteps the calibration concern from differencing two classifiers — and, in light of the F1 finding, it also sidesteps the problem of one of those classifiers being degenerate.

**The more portable idea is the intent/outcome split.** Training the same model with and without outcome features yields **I-VAEP** and **O-VAEP**. See [[intent-vs-outcome-valuation]].

This suggests a route around VAEP's reliability problem: if unreliability stems from goals being rare and heavily weighted, an intent model should be more stable. **Untested** — no vault source reports [[split-half-reliability]] for I-VAEP.

A caveat on the benchmark: the source's MAE/MedAE comparison tests **labelling strategies on a common architecture**, not this model against Decroos's.

## Ratings Over Time

[[player-rating-time-series|Mendes-Neves et al. argue against the final aggregation step]] — summing a season into one number discards form, development, and style change. Their [[performance-volatility|volatility metrics]] and [[player-development-curve|development curve]] use per-game series instead.

This partly reframes the reliability critique below. Within-season movement is noise *if* measurement variance and signal *if* genuine form — a distinction no vault source has settled.

## Comparison with xT

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] — including VAEP's own authors — compare the two directly:

| | VAEP | [[expected-threat\|xT]] |
|---|---|---|
| Game state | Last 3 actions + context | Ball's pitch zone only |
| Actions valued | All 21 types | Ball progression only |
| Models risk (conceding) | Yes, nominally | No |
| Values finishing | Yes | No |
| [[interpretability]] | Low | High |
| [[split-half-reliability\|Reliability]] | **ρ = 0.25** | **ρ = 0.89** |
| Correlates with | Goals/90 (ρ=0.41) | Assists/90 (ρ=0.53) |

The "models risk" row now carries the qualification above — the mechanism is present, its classifier is not contributing signal at moderate data scale.

**Where VAEP wins.** The risk–reward trade-off of backward passes into one's own box; counterattack initiation, which xT scores near zero; short dribbles inside the box that cross no zone boundary.

**Where VAEP loses.** Ratings replicate poorly across random halves of a season ($\rho = 0.25$ against 0.89). Restricting VAEP to xT's action set only lifts this to 0.59 — the instability comes from the richer state representation itself. The chief culprit is that goals carry large values and are rare: three goals can double or halve a defender's rating.

Note that **the same rarity drives both problems** — the reliability failure and the F1 failure are two symptoms of one cause. VAEP is built on an event too rare to support the model it carries.

## Relation to Basketball's EPV

[[martingale-epv]] answers the same question from [[optical-tracking-data]] with a Bayesian [[stochastic-process]] model. Being a genuine conditional expectation it is a [[martingale]], so raw EPV changes average to zero, forcing a relative baseline (EPVA). VAEP's values carry no such constraint and can be summed directly — at the cost of that consistency guarantee.

## Player Ratings

$$rating(p) = \frac{90}{m} \sum_{a_i \in A_p^T} V(a_i)$$

Ratings decompose by action type to characterise style — Neymar's dribble value (0.16) against Coutinho's pass value (0.28).

## Relation to Expected Goals

[[expected-goals|xG]] is a special case: computing a shot's xG equals estimating $P_{scores}$ immediately before it. VAEP generalises this to all action types.

## Limitations

- Only values on-ball actions — pressing, marking, and off-ball movement are invisible.
- **The conceding classifier is degenerate at moderate data scale** (F1 = 0.000 on 45 matches), so defensive value is largely notional. See above.
- Structurally favours attackers; Virgil van Dijk ranks 81st — while topping both of [[duel-skill-rating|Shelopugin's duel tables]], which shows the information exists in event data unmodelled.
- Low [[split-half-reliability|reliability]] ($\rho = 0.25$), from the same goal-rarity cause.
- Cross-league and cross-club comparison is difficult.
- Low interpretability: no straightforward explanation for a given action's value.
- Defensive actions are mis-valued because [[event-stream-data|event data]] lacks context to judge tackles and interceptions — a *data* limitation noted independently by Mendes-Neves et al.

## See Also

- [[action-valuation]] · [[defensive-valuation]] · [[vdep]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[probability-calibration]]
- [[intent-vs-outcome-valuation]] · [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]]
- [[expected-possession-value]] · [[expected-threat]] · [[martingale-epv]] · [[expected-goals]]
- [[spadl]] · [[split-half-reliability]] · [[event-stream-data]] · [[duel-skill-rating]]
- [[gradient-boosting]] · [[random-forest]] · [[markov-game]] · [[martingale]]
- [[action-valuation-frameworks-compared]]
- [[evaluating-football-player-actions|VAEP Source Summary]] · [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Comparison Summary]]
- [[football-performance-time-series|Valuing Players Over Time Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
