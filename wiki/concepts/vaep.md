---
title: "VAEP (Valuing Actions by Estimating Probabilities)"
type: concept
tags: [machine-learning, sports-analytics, statistics, evaluation, player-evaluation, probabilistic-classification, markov-model, action-valuation, defensive-valuation, reliability, regression, random-forest, time-series, class-imbalance]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/football-performance-time-series.md, raw/papers/football_defence_evaluation.md]
confidence: 0.95
provenance:
  extracted: 80%
  inferred: 10%
  generated: 7%
  imported: 0%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-27
---

# VAEP (Valuing Actions by Estimating Probabilities)

VAEP ([[evaluating-football-player-actions|Decroos et al., 2019]]) assigns a value to every on-ball action based on its impact on the probability of scoring and conceding in the near future. It is the canonical *action-based* approach in the [[action-valuation]] taxonomy.

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

## ⚠️ Does the Defensive Half Work?

The most consequential open question about VAEP, and the vault's account of it has needed correcting once already.

[[football-defence-evaluation-vdep|Toda et al. (2022)]] re-implement VAEP on 45 J-League matches and evaluate both classifiers with **F1** alongside AUC and Brier:

| Classifier | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{scores}$ | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ | 0.701 | 0.003 | **0.000** |

The vault previously recorded this as the defensive half being *empirically inert*. **That reading uses the wrong evidence.**^[generated: this objection is raised here, not by any held source]

**F1 requires hard predictions**, thresholded conventionally at 0.5. A well-calibrated classifier on a 0.23% base rate emits probabilities overwhelmingly below 0.05, crosses 0.5 almost never, and scores F1 = 0.000 **by construction** — however well it discriminates. This one shows AUC 0.701, which is not chance, and the best calibration in the comparison.

And **VAEP never thresholds.** It computes $\Delta P_{concedes}$, a difference of two probabilities. Hard classification appears nowhere in the pipeline, so the reported failure is of a use case that does not exist. See [[class-imbalance-evaluation]].

### What still indicts it

The reframe does not exonerate the model, because there is independent evidence: **VAEP correlates $\approx 0$ with goals conceded** — $r = -0.098$ across a season, $-0.040$ within a match — despite being constructed from a conceding classifier. That is a failure of the quantity VAEP actually uses, and no thresholding artefact explains it.

So the conclusion may be right while the headline diagnostic is wrong, which is a worse position than either being simply right or simply wrong.

### The test that would settle it

Report the **standard deviation of $\Delta P_{concedes}$ relative to $\Delta P_{scores}$**, and ablate the defensive term to see whether player rankings move. If the term is near-constant, VAEP reduces in practice to its offensive half — which would be a stronger and better-founded claim than "F1 = 0.000". One number and one ablation, neither run. See [[vaep-conceding-classifier]].

Note the scope caveat regardless: Decroos et al. trained on 8.5M actions, roughly 75× more conceding events than the 45-match sample, and never reported F1 or PR-AUC.

## Data Foundation

VAEP is built on [[spadl]] representations of [[event-stream-data]]. The fixed 9-attribute action format makes it possible to construct the fixed-length feature vectors the probability estimators require.

## Probability Estimation

Two [[probabilistic-classification|probabilistic classifiers]] ([[gradient-boosting|CatBoost]]) estimate $P_{scores}$ and $P_{concedes}$ from features of the previous 3 actions. CatBoost outperforms Logistic Regression, [[random-forest|Random Forest]] and XGBoost on both Brier score and ROC AUC.

Because action values are computed by **summing and subtracting** probabilities, [[probability-calibration|calibration]] is essential — and, per the section above, not sufficient. A classifier predicting the base rate is perfectly calibrated and contributes nothing.

Note that CatBoost's win here is **sample-size dependent**: on [[optimal-decisions-shot-taking-situations|Yeung & Fujii's]] 2,575 shots, both tree ensembles came last. See [[gradient-boosting]].

## Variants: I-VAEP and O-VAEP

[[football-performance-time-series|Mendes-Neves, Meireles & Mendes-Moreira]] build a modified VAEP departing from the original in three ways:

| | Original | Mendes-Neves et al. |
|---|---|---|
| Target | Two probabilities | One continuous label on $[-1, 1]$ |
| Framing | [[probabilistic-classification\|Classification]] | Regression |
| Label horizon | Next $k = 10$ actions | Time-decayed (capped at 1 min), floored at last 5 actions |
| Learner | [[gradient-boosting\|CatBoost]] | [[random-forest\|Random Forest]] |
| Differencing | Consecutive states | **Lag of two actions** |

Collapsing two probabilities into one signed target means offensive and defensive value are no longer separately estimated. That loses independent inspection of each — but in light of the section above, it also sidesteps the problem of one of those classifiers possibly being degenerate.

**The more portable idea is the intent/outcome split.** Training the same model with and without outcome features yields **I-VAEP** and **O-VAEP**. See [[intent-vs-outcome-valuation]].

This suggests a route around VAEP's reliability problem: if instability stems from goals being rare and heavily weighted, an intent model should be more stable. **Untested** — no vault source reports [[split-half-reliability]] for I-VAEP.

## Ratings Over Time

[[player-rating-time-series|Mendes-Neves et al. argue against the final aggregation step]] — summing a season into one number discards form, development and style change.

This bears directly on the reliability critique below. Within-season movement is noise *if* measurement variance and signal *if* genuine form — and those two readings turn out to be **the same variance component**, not two phenomena. See [[within-season-variation-noise-or-signal]].

## Comparison with xT

[[on-ball-actions-football-xt-vs-vaep|Van Roy et al. (2020)]] — including VAEP's own authors — compare directly:

| | VAEP | [[expected-threat\|xT]] |
|---|---|---|
| Game state | Last 3 actions + context | Ball's pitch zone only |
| Actions valued | All 21 types | Ball progression only |
| Models risk (conceding) | Yes, nominally | No |
| Values finishing | Yes | No |
| [[interpretability]] | Low | High |
| [[split-half-reliability\|Reliability]] | **ρ = 0.25** | **ρ = 0.89** |
| Correlates with | Goals/90 (ρ=0.41) | Assists/90 (ρ=0.53) |

**Where VAEP wins.** The risk–reward trade-off of backward passes into one's own box; counterattack initiation, which xT scores near zero; short dribbles inside the box that cross no zone boundary.

**Where VAEP loses.** Ratings replicate poorly across random halves of a season. Restricting VAEP to xT's action set only lifts this to 0.59 — the instability comes from the richer state representation itself. The chief culprit is that goals carry large values and are rare.

Note that **the same rarity drives both problems**^[generated: the connection between the reliability failure and the conceding-classifier failure is drawn here, not by any source] — the reliability failure and the defensive-classifier failure are two symptoms of one cause. VAEP is built on an event too rare to support the model it carries.

## Relation to Basketball's EPV

[[martingale-epv]] answers the same question from [[optical-tracking-data]] with a Bayesian [[stochastic-process]] model. Being a genuine conditional expectation it is a [[martingale]], so raw EPV changes average to zero, forcing a relative baseline (EPVA). VAEP's values carry no such constraint and can be summed directly — at the cost of that consistency guarantee.

## Player Ratings

$$rating(p) = \frac{90}{m} \sum_{a_i \in A_p^T} V(a_i)$$

Ratings decompose by action type to characterise style — Neymar's dribble value (0.16) against Coutinho's pass value (0.28).

## Limitations

- Only values on-ball actions — pressing, marking and off-ball movement are invisible.
- **The defensive term's contribution is unestablished** — see above. Not "zero", but unmeasured in the form that matters.
- Structurally favours attackers; van Dijk ranks 81st — while topping both of [[duel-skill-rating|Shelopugin's duel tables]], which shows the information exists in event data unmodelled.
- Low [[split-half-reliability|reliability]] (ρ = 0.25), from the same goal-rarity cause — though how much of that is metric noise versus genuine player inconsistency is itself open.
- Cross-league and cross-club comparison is difficult.
- Low interpretability.
- Defensive actions are mis-valued because [[event-stream-data|event data]] lacks context to judge tackles and interceptions.

## See Also

- [[vaep-conceding-classifier]] · [[within-season-variation-noise-or-signal]] — open questions on this page's two weakest points
- [[action-valuation]] · [[defensive-valuation]] · [[vdep]] · [[class-imbalance-evaluation]] · [[rare-event-proxy-targets]]
- [[intent-vs-outcome-valuation]] · [[player-rating-time-series]] · [[performance-volatility]] · [[split-half-reliability]]
- [[expected-possession-value]] · [[expected-threat]] · [[martingale-epv]] · [[expected-goals]] · [[spadl]]
- [[gradient-boosting]] · [[random-forest]] · [[probability-calibration]] · [[markov-game]]
- [[action-valuation-frameworks-compared]]
- [[evaluating-football-player-actions|VAEP Summary]] · [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
