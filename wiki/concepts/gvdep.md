---
title: "GVDEP (Generalized Valuing Defense by Estimating Probabilities)"
type: concept
tags: [defensive-valuation, sports-analytics, off-ball, action-valuation, proxy-target, gradient-boosting, model-selection, evaluation, optical-tracking-data]
sources: [raw/papers/defensive_player_location_analysis.md, raw/papers/football_defence_evaluation.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# GVDEP (Generalized Valuing Defense by Estimating Probabilities)

[[generalized-vdep-euro-location-analysis|Umemoto, Tsutsui & Fujii's (2022)]] generalisation of [[vdep|VDEP]], replacing its arbitrary weighting constant with score-scaled weights derived from [[vaep|VAEP]].

## The Fix

VDEP combines two probabilities:

$$V_{vdep}(s_i) = P_{gains}(s_i) - C \cdot P_{attacked}(s_i)$$

where $C$ came from the **frequency ratio** of the two events. That encodes how *often* each happens and says nothing about how much each *matters* — the vault's [[model-selection|standing objection]] to it, and one its own authors called controversial.

GVDEP replaces $C$ with two weights, each the mean VAEP value at the moments the corresponding event occurred:

$$V_{gvdep}(s_i) = w_{gains}\,\Delta P_{gains}(s_i) - w_{attacked}\,\Delta P_{attacked}(s_i)$$

$$w_{gains} = \frac{1}{|Ev_{gains}|}\sum_{j \in Ev_{gains}} \text{sign}(Team_j)\,V_{vaep}(s_j)$$

The first term becomes the change in scoring-or-conceding probability *after gaining the ball*; the second, the change *after being attacked*. Both are now on a **score scale** rather than a frequency scale.

Note what this costs: GVDEP now depends on VAEP's $P_{scores}$ and $P_{concedes}$, whose F1 scores here are **0.10–0.13 and 0.08–0.15**. A principled weight computed from a weak model is better-motivated than an arbitrary constant but not necessarily better-estimated.

## The n_nearest Finding

The paper's most portable contribution, and a rare **sensitivity analysis** in this literature. Sweeping the number of nearest attackers and defenders from 0 to 11:

| Prediction | F1 behaviour |
|---|---|
| **Ball gain** | 0.16 → ~0.31, **flat after 3–4 players** |
| Being attacked | ~0.44 → 0.46, flat throughout |
| Scores | ~0.10 → 0.13, no gain |
| Concedes | **0.15 → 0.08 — gets worse** |

**Ball gain is the only prediction needing player positions at all**, and it needs three or four, not twenty-two. Recovering the ball is a local contest; being attacked and conceding are properties of the broader configuration that the ball's own position already carries.

The concedes row is the interesting one and is not the authors' emphasis: **more player information makes that classifier worse.** With 186 positives in 100,328 events, extra dimensions are noise. The same small-data overfitting appears in [[xsot|Yeung & Fujii's]] shot data, where raw player coordinates performed below shooter features alone. See [[gradient-boosting]].

## Why Broadcast Data Matters

VDEP assumed full 22-player tracking. GVDEP uses **StatsBomb open data from broadcast video frames**, where the median scene shows 15 players and the quartiles are 11 and 18. Events with no visible players are dropped — 12,262 of 112,590 in Euro 2020, and a striking 28,218 of 61,433 in Euro 2022.

Combined with the n_nearest result, this makes the method available to anyone with public data rather than a tracking licence — the same democratising motive as [[obso|Spearman's]] deliberate minimisation of data requirements.

## Results

Across the last 16 of Euro 2020:

- **The gain/attacked trade-off replicates** ($r = -0.757$, $p = 0.001$) — teams that recover more concede more territory. VDEP found the same in the J-League, so this now holds across two continents and both a domestic league and an international tournament.
- **GVDEP correlates 0.993 with its attacked term alone**, because $|w_{attacked}| = 0.021$ against $|w_{gains}| = 0.011$. The authors flag it: the metric is nearly a monotone function of one component.
- **No significant correlation with concedes** ($r = -0.265$, $p = 0.321$), defended as measuring process rather than outcome.

Italy won with the best attacked-value and a *low* gain-value — they did not try to win the ball back. England scored well on both and conceded nothing to the round of 16.

## Where It Sits

| | [[vdep]] | **GVDEP** |
|---|---|---|
| Weighting | $C$ from event frequency | **VAEP at the event** |
| Scale | Frequency | **Score** |
| Observation assumed | All 22 players | **Broadcast frames, partial** |
| Data | J-League, men | **Euro 2020 men + Euro 2022 women** |
| Unit | Team | **Team** |
| Dominated by one term | Not reported | **Yes, $r = 0.993$** |

**Still team-level.** GVDEP does not individuate defensive credit, and the vault previously implied the Umemoto line would. The counterfactual-positioning work that might is Umemoto & Fujii (2023), a different and unheld paper. See [[defensive-valuation]].

## Limitations

- **Dominated by the attacked term**, acknowledged.
- **Weights derive from weak classifiers** — scores and concedes F1 of 0.08–0.15.
- $k = 5$ and $k' = 10$ inherited without re-examination. See [[free-parameters-load-bearing]].
- **Goalkeeping explicitly excluded**, which misprices teams defending by shot-stopping — Belgium and the Czech Republic score poorly despite few concedes.
- No [[split-half-reliability|reliability]] figure.
- Nearly half of Euro 2022 events discarded for having no visible players, which may not be missing at random — wide shots and set pieces are plausibly over-represented among the discards.

## See Also

- [[vdep]] · [[defensive-valuation]] · [[vaep]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[model-selection]] · [[free-parameters-load-bearing]] · [[gradient-boosting]] · [[obso]]
- [[rikuhei-umemoto]] · [[keisuke-fujii]] · [[kazushi-tsutsui]]
- [[generalized-vdep-euro-location-analysis|Source Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
