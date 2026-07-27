---
title: "Expected Possession Value of Control and Duel Actions (Shelopugin)"
type: source_summary
tags: [summary, sports-analytics, action-valuation, duel-analysis, player-evaluation, event-stream-data, discounting, ranking-system, transfer-prediction, recruitment, selection-bias, positive-unlabeled-learning, gradient-boosting, predictive-validity]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.8
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Expected Possession Value of Control and Duel Actions for Soccer Player's Skills Estimation

**Author:** [[andrei-shelopugin]] (sole author, independent researcher)
**Status:** Preprint. Data current to 1 June 2024.

A set of extensions to event-data [[expected-possession-value|EPV]] for soccer, plus a downstream model that forecasts the resulting player metric for the *next* season under a hypothetical transfer. The framing throughout is the **selection problem**: narrowing a scouting pool, not describing a completed season.

## The Four Contributions

1. **A [[temporal-discounting|decay effect]]** — actions closer in time to a shot receive more credit, via a discount factor $\gamma = 0.95$ per second.
2. **[[possession-risk|Possession risk]] handled properly** — the penalty for conceding is decayed by elapsed time and extends across an unbounded number of subsequent possessions rather than exactly one.
3. **[[symmetrical-duel-valuation|Duel valuation]]** — aerial and ground duels are assigned possession value, and individual [[duel-skill-rating|duel skill]] enters the model as a feature.
4. **[[transfer-performance-prediction|Next-season forecasting]]** — the season metric is predicted for a hypothetical destination club, using [[league-strength-rating|Glicko-2 club and league ratings]].

All four are built on [[event-stream-data|event data]] only — no tracking.

## Definitions That Do Work

The paper's definitions are load-bearing rather than incidental:

- **Control actions** — pass, shot, dribble, carry, free kick, goal kick, penalty, corner, throw-in. Defined as the actions where *it is unambiguous who has the ball*.
- **Symmetrical duels** — aerial and ground duels, where it is not. A dribble is explicitly *not* symmetrical.
- **Possession** — an uninterrupted series of control actions by one team. A touch that does not lead to a control action does not end the opponent's possession; a save leading to a corner does not either. One possession can therefore contain several shots.
- **[[effective-playing-time|Effective playing time]]** — ball-in-play time only, excluding VAR reviews, substitutions, and set-piece dead time. All metrics are normalised on this rather than clock time.

The control/duel split is the paper's organising move: it partitions events by *whether possession is attributable*, and then builds separate machinery for each side.

## The Target Variable

Possession value for control actions, without decay, is the probability that some future shot **in the same possession** yields a goal:

$$PV(c_i) = 1 - \prod_j (1 - xG_j)[t_j \ge t_i][s_i = s_j]$$

The complement-product form accounts for a later shot scoring after an earlier one missed. Only *future* shots count.

The author rejects the binary "did this possession end in a goal" target explicitly, on the grounds that goals are too rare and the model would overfit — choosing accumulated [[expected-goals|xG]] instead.

**With decay** (eq. 2), each shot's contribution is multiplied by $\gamma^{(t_j - t_i)}$. **With risk** (eq. 3), the opponent's decayed possession value across subsequent possessions is subtracted.

The interpretation of $\gamma$ is explicitly *not* the reinforcement-learning one. The paper is not claiming earlier goals are worth more; it is claiming that **an action which does not produce a shot soon probably did not advance the attack**. Consequently $\gamma$ is offered as a tunable stylistic parameter — 0.9 for a vertical attacking philosophy, 0.99 for possession-heavy *tiki-taka*.

## Duels: The Interesting Part

Duels break the possession-attribution assumption that every framework in this vault relies on. The paper's handling:

**Assigning value.** A duel inherits the possession value of the first control action that follows it, signed by whether the same team retained possession (eq. 4). Chains of consecutive duels recursively inherit the same value.

**The reward asymmetry.** A pass into an aerial duel initially looks like a *negative* action — the passer has created a 50/50. But that is only right if the duel really is 50/50. The fix is to feed the *probability of winning the duel* into the EPV model, so a pass to a dominant aerial target is rewarded more than the same pass to a weak one.

**Avoiding the circularity.** When valuing the duel-winner's own action, the model deliberately uses the **average**-player duel model rather than the individual one — otherwise a strong duellist would be penalised for having a high expected value to beat. This gives three separate models: $EPV$, $EPV^{avg}_{duel}$, $EPV^{ind}_{duel}$.

**Five reward scenarios** (eqs. 5–6) cover: possession retained, turnover (penalised twice — for the loss and for the opponent's resulting chance), goal scored, end of half (zero), and duel created.

## Implementation

Six [[gradient-boosting|LightGBM]] models for EPV — open play, set pieces, and average/individual variants for aerial and ground duels — plus separate xG models for set pieces and open play.

Two recurring tricks:

- **[[sample-weighting|Frequency-weighted loss]].** Both the xG and duel models divide each instance's loss by the number of times that player appears in the training set, so that heavily-represented star players do not dominate. Applied as a custom `objective` (log-loss for classification, MSE for regression).
- **Deliberate feature exclusion.** Score, competition, and team are omitted from xG *because* they correlate with player skill — the goal is an average-player model.

## Duel Ratings

Rather than win percentage — which the paper argues converges to ~50% for everyone, since coaches match strong players against strong opponents — duel skill is rated with a **[[duel-skill-rating|modified Glicko-2]]**.

The modification adds an advantage term $a$ to the expected-score calculation for the defender:

$$\mu' = \mu + \phi'^2 g(\phi_j)(s_j - E(\mu + a, \mu_j, \phi_j))$$

$a$ comes from a separate LightGBM model predicting duel outcome from *context only* — duel and pass coordinates, pass type, number of opponents — with all skill-related features excluded.

**Two leakage problems** are identified and addressed: defending teams have a structural edge in aerial duels, and centre-backs both excel aerially and contest more duels defensively, which would depress their ratings. The fix is six position categories, with the context model trained on same-category matchups only.

The paper also fixes a definition of *winner*: fouled player wins; else first touch wins; else the team gaining possession wins.

## PCR and Next-Season Prediction

**Pass Carry Reward** ([[pass-carry-reward|PCR]]) sums $\Delta EPV$ over a player's passes and carries across a season, normalised per 60 minutes of effective time.

The prediction model uses **~600 features** in five groups: player attributes; prior-season performance (plus 3- and 5-season averages); share of team output; league style (e.g. league mean PCR); and team/league strength.

**Cross-league comparability** comes from a Glicko-2 team rating, with league strength defined as the mean rating of its top *n* clubs — allowing features like "difference between old and new league rating". Only ratings available at season start are used.

**Two auxiliary problems:**

- *Who stays in the data.* The training set requires ≥100 minutes in both seasons, which cannot be known in advance, so a second model predicts the probability of clearing that bar next season — using contract length from the EA Sports FC dataset as a feature.
- *[[positive-unlabeled-learning|Presence-only data]].* Transfers are not random: a move to a stronger club signals perceived upside, a move down signals decline. Predictions are therefore biased optimistic upward and pessimistic downward. The correction is a heuristic shrinkage, $PCR_{adj} = PCR \cdot 0.8^{(\Delta ratings + p_l)}$.

## Results

Against a persistence baseline (last season's PCR as the prediction):

| Sample | Baseline RMSE | Model RMSE |
|---|---|---|
| All data (>100 min) | 0.053 | **0.033** |
| Same team, same league | 0.050 | **0.032** |
| New team, new league | 0.061 | **0.037** |

The baseline degrades as the player moves further from their prior context; so does the model, but less. This is a genuine **player-level** [[predictive-validity|predictive validity]] result, which the vault has otherwise only at team level.

Case-study output includes per-club shortlists for Manchester City, Barcelona, Milan and Brighton — Brighton chosen for its data-led recruitment reputation.

### The Donnarumma Case Study

The appendix demonstrates why individual duel skill matters. Under Pioli, Donnarumma's instruction was to play long when pressed, usually into a duel. Comparing his long passes to Ibrahimović against those to Leão:

| Target | Duel | n | Won (a priori) | Won (skill-adjusted) |
|---|---|---|---|---|
| Leão | aerial | 35 | 40.5% | 35.8% |
| Ibrahimović | aerial | 28 | 39.2% | **61.1%** |

The *situations* were comparably difficult, and Ibrahimović faced marginally stronger opponents — yet the outcomes diverge sharply. A duel-blind EPV model scores the two passes almost identically (0.00075 vs 0.00077); the skill-aware model separates them (0.00069 vs 0.00135).

## Assessment

**Strengths.** The decay treatment is a real improvement on hard action-count windows, and the duel work addresses a blind spot shared by every other valuation framework in this vault. The frequency-weighted loss and the leakage analysis in the duel ratings show unusual care about *which* biases event data induces.

**Weaknesses and open questions.**

- **No validation of PCR itself.** The author states plainly that there is no mathematical way to prove EPV-based metrics track player skill, and proposes only two future routes: expert review, or checking shortlists against actual top-club transfers. Neither is executed. The RMSE results validate the *forecast*, not the *metric*.
- **$\gamma = 0.95$ is unjustified empirically.** Framed as a tunable preference, but no sensitivity analysis is given.
- **The presence-only correction is a heuristic.** The 0.8 base and the $\Delta ratings$ scaling are asserted, not derived; the author flags this as future work.
- **Single-author preprint, no peer review**, and the underlying data provider is never named — though the Wyscout-style duel taxonomy suggests it.
- **Position drift unmodelled.** Explicitly acknowledged: the model mispredicts when a centre-forward becomes a winger at the new club.
- **Reward attribution for accurate passes remains unsolved** — the duel machinery only splits credit when the pass leads to a contest.

## See Also

- [[pass-carry-reward]] · [[symmetrical-duel-valuation]] · [[duel-skill-rating]]
- [[temporal-discounting]] · [[possession-risk]] · [[effective-playing-time]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[positive-unlabeled-learning]]
- [[expected-possession-value]] · [[action-valuation]] · [[expected-goals]]
- [[glicko-rating-system]] · [[bradley-terry-model]]
- [[recruitment]] · [[selection-bias]] · [[predictive-validity]]
- [[action-valuation-frameworks-compared]]
