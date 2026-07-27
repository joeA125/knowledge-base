---
title: "Transfer Performance Prediction"
type: concept
tags: [transfer-prediction, recruitment, sports-analytics, player-evaluation, regression, counterfactual, predictive-validity, selection-bias, positive-unlabeled-learning]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.75
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Transfer Performance Prediction

Forecasting what a player will produce **next season, at a specified destination club**. Distinct from valuation (what did they produce?) and from forecasting (what happens next in this match?), and it is the question [[recruitment]] actually needs answered.

The vault now holds two structurally different attacks on it.

## Two Approaches

| | **Regression** ([[epv-control-duel-skills-football\|Shelopugin]]) | **[[counterfactual-simulation\|Generative simulation]]** ([[scoutgpt]]) |
|---|---|---|
| Object modelled | Season aggregate | Event sequence |
| Destination encoded as | [[league-strength-rating\|Club/league rating features]] | Explicit lineup conditioning |
| Output | Predicted [[pass-carry-reward\|PCR]] | Simulated [[vaep]] over generated episodes |
| Data needed | Season-level history | Full event streams for the destination squad |
| Tactical fit | Via aggregate strength only | Via actual teammates |
| Handles selection bias | Explicitly, heuristically | Not addressed |
| Cost | Modest | Substantial |

They are complementary rather than competing. The regression approach answers "how well does this player's level transfer to that standard of competition?" The generative approach answers "how does this player interact with those specific teammates?" A club wants both, and neither subsumes the other.

## The Regression Formulation

Predict next-season PCR from ~600 features in five groups:

1. **Player attributes** — age, height, position.
2. **Prior performance** — PCR, xG, goals, minutes, plus 3- and 5-season averages.
3. **Share of team output** — e.g. the player's proportion of team xG while on the pitch. This matters because a good player at a weak club has poor absolute numbers; share separates the player from the team.
4. **League style** — league mean PCR and similar. Attacking leagues inflate everyone's numbers.
5. **Team and league strength** — [[league-strength-rating|Glicko-2 ratings]] of old and new club and league, and mean opponent rating.

Group 3 is the underrated one. It is the cheapest available correction for context confounding, and it does not require any of the rating machinery.

## Results, and What They Establish

Against a persistence baseline (predict this season's PCR for next season):

| Sample | Baseline RMSE | Model RMSE | Improvement |
|---|---|---|---|
| All (>100 min) | 0.053 | 0.033 | 38% |
| Same team, same league | 0.050 | 0.032 | 36% |
| New team, new league | 0.061 | 0.037 | 39% |

Two things stand out. The baseline **degrades as the player moves** — persistence works worst exactly where recruitment needs it most — and the model's improvement holds roughly constant across the movement categories rather than collapsing under distribution shift.

This is the vault's first **player-level** [[predictive-validity]] evidence. Everything prior ([[hpus]], [[lpv]] versus next-match xG) is team-level, and the synthesis explicitly flags that team-level results do not license player-level conclusions. This partially closes that gap.

Partially, because what is predicted is the metric's own future value, not an independent outcome. A metric can be self-predictable while measuring the wrong thing.

## Why This Is Hard

**Selection.** Transfers are decisions made by people forecasting the same quantity. See [[positive-unlabeled-learning]] — the observed sample of upward moves is filtered by scouting judgement, so naive predictions are optimistic for step-ups and pessimistic for step-downs.

**Attrition.** The training set needs the player to appear next season at all. A separate model predicts the probability of clearing a 100-minute threshold, using contract length among its features, since players may retire, be injured, be frozen out, or drop to a league outside the dataset.

**Role change.** The paper's clearest acknowledged failure: a centre-forward who becomes a winger at the new club is mispredicted, because the target is unconditional on role. The proposed fix is to predict PCR *conditional on the expected position*, which is not implemented.

**Price is absent.** No vault source models fee or wages, so none of this yields value-for-money — a persistent limitation noted on [[recruitment]].

## Practical Output

The paper's deliverable is a **destination-conditioned shortlist**: rankings of players by predicted PCR *if they signed for Manchester City*, or Barcelona, or Milan, or Brighton. Rankings differ meaningfully by destination, which is the point — a player is not equally suited to every club.

This is the correct shape for a scouting tool. It is a filter that narrows a market to a reviewable list, not a signing recommendation, and the author frames it that way throughout.

## See Also

- [[recruitment]] · [[counterfactual-simulation]] · [[scoutgpt]]
- [[pass-carry-reward]] · [[league-strength-rating]]
- [[positive-unlabeled-learning]] · [[selection-bias]] · [[predictive-validity]]
- [[player-development-curve]] · [[performance-volatility]]
- [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Source Summary]]
