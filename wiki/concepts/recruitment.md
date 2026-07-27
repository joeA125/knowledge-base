---
title: "Recruitment"
type: concept
tags: [sports-analytics, recruitment, transfer-prediction, player-evaluation, player-development, volatility, evaluation, reliability, predictive-validity, counterfactual, selection-bias, positive-unlabeled-learning]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Recruitment

Recruitment is the decision problem most football analytics is ultimately built to serve: **should this club sign this player, at this price, now?** It is worth treating as a concept in its own right because it imposes requirements that differ sharply from those of the modelling tasks in [[action-valuation-frameworks-compared]] — and because several metrics that look strong on their own terms serve it badly.

## What the Decision Actually Requires

A signing is a forecast under [[counterfactual-simulation|transfer of context]]. The question is never "how much value did this player produce?" but "how much will they produce **for us**, **next season**, **given what we pay**?" That decomposes into four sub-questions, only the first of which most metrics address:

| Sub-question | What it needs | Vault coverage |
|---|---|---|
| How good are they *now*? | A [[reliability\|reliable]] point estimate | Well covered — [[expected-threat\|xT]], [[vaep]] |
| Will that **transfer** to our team and league? | Context-invariance, or explicit re-simulation | Two approaches now — see below |
| Are they **improving or declining**? | A longitudinal view | [[player-development-curve\|PDC]], [[player-rating-time-series\|rating series]] |
| How **risky** are they? | A dispersion measure, not a mean | [[performance-volatility\|Volatility]] |

The second row was the weak point of the entire field, and remains the least settled. Observational value is confounded with the team, league, and role a player was already in, and almost every framework silently reports it as if it were a property of the player.

## Two Attacks on the Transfer Question

The vault now holds two structurally different approaches, and they are complementary rather than competing. See [[transfer-performance-prediction]] for the full comparison.

**Generative re-simulation.** [[scoutgpt]] conditions event generation on an explicit lineup, so substituting a player and regenerating estimates their output among *those specific teammates*. Expensive, and requires full event streams for the destination squad, but it is the only approach that models tactical interaction.

**Regression on context features.** [[epv-control-duel-skills-football|Shelopugin]] predicts next-season [[pass-carry-reward|PCR]] from ~600 features including [[league-strength-rating|Glicko-2 club and league ratings]], mean opponent rating, league style, and — importantly — the player's *share* of team output. Cheap, and applicable to any player in any rated league.

The share-of-team-output feature deserves particular note. A good player at a weak club has poor absolute numbers; expressing contribution as a proportion of what the team generated while they were on the pitch is the cheapest available correction for context confounding, and needs none of the rating machinery.

The evidence is encouraging but limited. Against a persistence baseline, prediction error falls by roughly 38%, and — the useful part — the improvement holds up for players changing both club and league (0.061 → 0.037 RMSE), which is exactly the case where naive extrapolation degrades most.

## Why Reliability Dominates Here

For recruitment specifically, [[split-half-reliability]] is close to the decisive criterion, and this reverses the usual ranking of metrics.

[[vaep|VAEP]] is the richer, more sophisticated model — context-aware, risk-modelling, valuing all 21 action types. Yet its player ratings replicate at $\rho = 0.25$ across halves of a season, against [[expected-threat|xT]]'s 0.89. A rating that unstable cannot support a multi-million-pound decision, however well-motivated the model behind it.

The lesson generalises: **sophistication that buys sensitivity is a liability when the output is a season-long judgement about a person.** VAEP's context-sensitivity is exactly right for analysing a passage of play and exactly wrong for ranking transfer targets.

This is the main unknown about PCR. It is a rich, [[possession-risk|risk-modelling]] metric of the VAEP family rather than the xT family, and no reliability figure is reported — so whether it inherits VAEP's instability is untested and is the single most useful missing number in that work.

## The Age Dimension and Market Inefficiency

Two players at identical current output are different assets if one is 23 and the other 30. The [[player-development-curve|PDC]] locates a player relative to their expected peak — roughly 25 to 27 — turning a static rating into an appreciating or depreciating position.

[[football-performance-time-series|Mendes-Neves et al.]] draw an explicit **market-inefficiency** argument from this. Transfer valuation falls sharply once a player passes the nominal peak age, because pricing applies the population curve to every individual. Players who actually peak later — the source names Tiago, Aritz Aduriz, Joaquín — are therefore systematically underpriced.

The inefficiency exists *because* the average curve is widely believed. A club willing to evaluate the individual trajectory rather than the population one buys real production cheaply. This is one of the few places in the vault where an analytics finding translates into a directly actionable strategy rather than a better description.

## Risk, Not Just Expectation

Squad building is a portfolio problem, and mean output is the wrong single summary. Two players with identical averages differ if one delivers consistently and the other alternates decisive with anonymous.

[[performance-volatility|Volatility metrics]] — particularly the downside-only variants, residualised against rating level — make this comparable. The practical uses are to pair high-ceiling volatile players with consistent ones rather than optimising every slot for mean, and to treat consistency as genuine value at equal expected contribution.

The untested extension flagged by the source is the most valuable one: **volatility conditioned on opposition strength**, separating players who raise their level against good teams from those who accumulate against weak ones. [[league-strength-rating|Mean opponent rating]] supplies the missing ingredient for exactly this test, and nobody has run it.

## The Population You Cannot Measure

Every metric discussed here requires a substantial sample — minutes thresholds, games-played minimums, a 40-game window with a 20-game floor. This creates a structural [[selection-bias|selection]] problem specific to recruitment:

**The players hardest to evaluate are exactly the ones excluded.** Young players, fringe players, players recently transferred, and players from lower divisions all fail the sample thresholds. These are precisely the recruitment targets where good information would be most valuable, and where clubs consequently still rely on scouts.

Analytics is therefore strongest where it is least needed — established players in elite leagues, whose quality is already widely known and priced — and weakest where the market inefficiency is largest. Lowering thresholds does not solve this; it produces unstable ratings instead of missing ones.

A second, sharper version of the problem sits inside transfer forecasting itself. Observed transfers are not a random sample: a move upward signals that someone judged the player capable of it. Training on them makes predictions optimistic for step-ups and pessimistic for step-downs, and the effect cannot be removed by adding features about the destination. See [[positive-unlabeled-learning]].

## Practical Ordering

Given the above, a defensible metric stack for a recruitment decision:

1. **Level** — a stable rating ([[expected-threat|xT]], or [[intent-vs-outcome-valuation|I-VAEP]] if the intent/outcome split is available).
2. **Trajectory** — position on the [[player-development-curve|PDC]] and the direction of the [[player-rating-time-series|long-term series]].
3. **Risk** — [[performance-volatility|downside volatility]], residualised against rating.
4. **Context transfer** — [[transfer-performance-prediction|regression on league and club strength]] to shortlist broadly, then [[counterfactual-simulation|re-simulation]] under the target lineup for the final few, where data permits.
5. **Everything the data cannot see** — off-ball work, defensive context, temperament, medical. Still scouting's domain.

The ordering of step 4 reflects cost. Regression scales to a whole market; simulation does not. Used together, the first narrows and the second discriminates.

## Limitations

- **Offensive bias throughout.** Every valuation framework undervalues defenders, so an analytics-led process will systematically misprice them. Duel ratings ([[duel-skill-rating]]) are a partial counterweight, and notably rank van Dijk first where valuation frameworks rank him 81st and 142nd.
- **Price is exogenous.** No vault source models transfer fee or wages, so "value for money" cannot be computed from these metrics alone. This remains the largest gap in the chain from metric to decision.
- **Role change is unmodelled.** Forecasts assume the player occupies the same position at the new club; a centre-forward converted to a winger is mispredicted, and the fix — forecasting conditional on expected role — is proposed nowhere and implemented nowhere.
- **Team-level [[predictive-validity]] is not player-level.** The evidence that possession metrics outpredict goals is a *team-match* result. Shelopugin's next-season forecasts are the vault's first player-level evidence, but they predict the *metric's own future value* rather than an independent outcome — a weaker test, since a metric can be self-predictable while measuring the wrong thing.

## See Also

- [[transfer-performance-prediction]] · [[counterfactual-simulation]] · [[scoutgpt]]
- [[player-development-curve]] · [[performance-volatility]] · [[player-rating-time-series]]
- [[intent-vs-outcome-valuation]] · [[pass-carry-reward]] · [[league-strength-rating]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]] · [[positive-unlabeled-learning]]
- [[expected-threat]] · [[vaep]] · [[duel-skill-rating]]
- [[action-valuation-frameworks-compared]]
- [[football-performance-time-series|Source Summary]] · [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
