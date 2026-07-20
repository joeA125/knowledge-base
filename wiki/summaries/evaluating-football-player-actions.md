---
title: "Actions Speak Louder than Goals: Valuing Player Actions in Soccer — Source Summary"
type: source_summary
tags: [machine-learning, sports-analytics, statistics, evaluation, player-evaluation, event-stream-data, gradient-boosting]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Actions Speak Louder than Goals: Valuing Player Actions in Soccer

**Authors:** [[tom-decroos]], Lotte Bransen, Jan Van Haaren, [[jesse-davis]]
**Affiliations:** KU Leuven, SciSports
**Published:** 2019 (KDD 2019)

## Key Contribution

Introduces [[spadl]] (Soccer Player Action Description Language), a unified representation for [[event-stream-data]], and [[vaep]] (Valuing Actions by Estimating Probabilities), a framework that assigns a value to every on-ball action based on its impact on the probability of scoring and conceding. Unlike traditional metrics (goals, assists, [[expected-goals|xG]]), VAEP values all 21 action types, accounts for context, and captures longer-term effects up to 10 actions ahead.

## SPADL: Unified Action Representation

Addresses five data science challenges with vendor [[event-stream-data|event streams]] (Opta, Wyscout, StatsBomb): multiple objectives, inconsistent terminology, backward compatibility bloat, optional information snippets, and variable-length features. SPADL represents each action as a fixed tuple of 9 attributes (StartTime, EndTime, StartLoc, EndLoc, Player, Team, ActionType, BodyPart, Result) across 21 action types. A Python package converts vendor formats automatically.

## VAEP: Action Valuation Framework

For each game state $S_i = [a_1, \ldots, a_i]$, estimate scoring and conceding probabilities for the team in possession:

$$V(a_i) = \Delta P_{scores}(a_i) + (-\Delta P_{concedes}(a_i))$$

where $\Delta P_{scores}(a_i) = P_{scores}(S_i) - P_{scores}(S_{i-1})$. Positive values indicate actions that help the team; negative values indicate actions that hurt. Player ratings aggregate per 90 minutes: $rating(p) = \frac{90}{m} \sum_{a_i \in A_p^T} V(a_i)$.

## Probability Estimation

Two binary [[probabilistic-classification|probabilistic classifiers]] ([[gradient-boosting|CatBoost]]) trained on features from the previous 3 actions:
- **SPADL features:** Action type, result, body part, locations, time
- **Complex features:** Distance/angle to goal, distance covered, speed of play, possession change
- **Game context:** Goals scored, goals conceded, goal difference

Labels: whether a goal is scored/conceded in the next $k=10$ actions. CatBoost outperforms Logistic Regression, Random Forest, and XGBoost ([[probability-calibration|Brier]] 0.01376, AUC 0.7693 for scoring). Because action values sum and subtract probabilities, [[probability-calibration|calibration]] — not just discrimination — is essential.

## Key Results

### Player Ratings (2017/18 EPL)
VAEP top 10: Coutinho (0.899), Salah (0.817), De Bruyne (0.641), Hazard (0.636), Mahrez (0.635). De Bruyne, Hazard, and Mahrez don't appear in any traditional top-10 (goals, assists, goals+assists) despite being Premier League stars. Combined market value of VAEP top-10 (€1,110M) exceeds goals top-10 (€862M).

### Scouting Use Case
Young talent identification: Rashford, Alexander-Arnold, Dembélé in major leagues; Neres, Mount, Mbappé, de Jong in minor leagues — all subsequently validated by major transfers or awards.

### Playing Style Characterisation
Action-type breakdowns enable style comparison: Neymar is a superior dribbler; Coutinho/Dembélé are better passers. Ronaldo's shot value exceeds Hazard + Rashford combined.

### Quality vs Quantity
Messi is an outlier — high on both action quality (0.017 value/action) and quantity (82 actions/90min). Strikers (Kane, Salah) favour few high-value actions; midfielders (De Bruyne, Pogba) favour many moderate-value actions.

## Limitations

- Only values on-ball actions (not defensive positioning, pressing, off-ball movement)
- Cross-league comparison is difficult (easier to perform valuable actions in minor leagues)
- Cross-club comparison is difficult (easier to perform valuable actions at top clubs)
- Building trust with traditional scouts remains a challenge

## See Also

- [[vaep]]
- [[spadl]]
- [[expected-goals]]
- [[event-stream-data]]
- [[gradient-boosting]]
- [[probability-calibration]]
- [[game-state-reconstruction]]
