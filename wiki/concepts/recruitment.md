---
title: "Recruitment"
type: concept
tags: [sports-analytics, recruitment, player-evaluation, player-development, volatility, evaluation, reliability, predictive-validity, counterfactual, selection-bias]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.75
provenance:
  extracted: 40%
  inferred: 55%
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
| Will that **transfer** to our team and league? | Context-invariance, or explicit re-simulation | Thin — only [[scoutgpt]] attacks it |
| Are they **improving or declining**? | A longitudinal view | [[player-development-curve\|PDC]], [[player-rating-time-series\|rating series]] |
| How **risky** are they? | A dispersion measure, not a mean | [[performance-volatility\|Volatility]] |

The second row is the weak point of the entire field. Observational value is confounded with the team, league, and role a player was already in, and almost every framework silently reports it as if it were a property of the player.

## Why Reliability Dominates Here

For recruitment specifically, [[split-half-reliability]] is close to the decisive criterion, and this reverses the usual ranking of metrics.

[[vaep|VAEP]] is the richer, more sophisticated model — context-aware, risk-modelling, valuing all 21 action types. Yet its player ratings replicate at $\rho = 0.25$ across halves of a season, against [[expected-threat|xT]]'s 0.89. A rating that unstable cannot support a multi-million-pound decision, however well-motivated the model behind it.

The lesson generalises: **sophistication that buys sensitivity is a liability when the output is a season-long judgement about a person.** VAEP's context-sensitivity is exactly right for analysing a passage of play and exactly wrong for ranking transfer targets.

## The Age Dimension and Market Inefficiency

Two players at identical current output are different assets if one is 23 and the other 30. The [[player-development-curve|PDC]] locates a player relative to their expected peak — roughly 25 to 27 — turning a static rating into an appreciating or depreciating position.

[[football-performance-time-series|Mendes-Neves et al.]] draw an explicit **market-inefficiency** argument from this. Transfer valuation falls sharply once a player passes the nominal peak age, because pricing applies the population curve to every individual. Players who actually peak later — the source names Tiago, Aritz Aduriz, Joaquín — are therefore systematically underpriced.

The inefficiency exists *because* the average curve is widely believed. A club willing to evaluate the individual trajectory rather than the population one buys real production cheaply. This is one of the few places in the vault where an analytics finding translates into a directly actionable strategy rather than a better description.

## Risk, Not Just Expectation

Squad building is a portfolio problem, and mean output is the wrong single summary. Two players with identical averages differ if one delivers consistently and the other alternates decisive with anonymous.

[[performance-volatility|Volatility metrics]] — particularly the downside-only variants, residualised against rating level — make this comparable. The practical uses are to pair high-ceiling volatile players with consistent ones rather than optimising every slot for mean, and to treat consistency as genuine value at equal expected contribution.

The untested extension flagged by the source is the most valuable one: **volatility conditioned on opposition strength**, separating players who raise their level against good teams from those who accumulate against weak ones.

## The Population You Cannot Measure

Every metric discussed here requires a substantial sample — minutes thresholds, games-played minimums, a 40-game window with a 20-game floor. This creates a structural [[selection-bias|selection]] problem specific to recruitment:

**The players hardest to evaluate are exactly the ones excluded.** Young players, fringe players, players recently transferred, and players from lower divisions all fail the sample thresholds. These are precisely the recruitment targets where good information would be most valuable, and where clubs consequently still rely on scouts.

Analytics is therefore strongest where it is least needed — established players in elite leagues, whose quality is already widely known and priced — and weakest where the market inefficiency is largest. Lowering thresholds does not solve this; it produces unstable ratings instead of missing ones.

## Practical Ordering

Given the above, a defensible metric stack for a recruitment decision:

1. **Level** — a stable rating ([[expected-threat|xT]], or [[intent-vs-outcome-valuation|I-VAEP]] if the intent/outcome split is available).
2. **Trajectory** — position on the [[player-development-curve|PDC]] and the direction of the [[player-rating-time-series|long-term series]].
3. **Risk** — [[performance-volatility|downside volatility]], residualised against rating.
4. **Fit** — [[counterfactual-simulation|re-simulation]] under the target lineup, where data permits.
5. **Everything the data cannot see** — off-ball work, defensive context, temperament, medical. Still scouting's domain.

## Limitations

- **Offensive bias throughout.** Every valuation framework undervalues defenders, so an analytics-led process will systematically misprice them.
- **Price is exogenous.** No vault source models transfer fee or wages, so "value for money" cannot be computed from these metrics alone.
- **Team-level [[predictive-validity]] is not player-level.** The evidence that possession metrics outpredict goals is a *team-match* result and does not license the assumption that player ratings forecast individual future output.

## See Also

- [[player-development-curve]] · [[performance-volatility]] · [[player-rating-time-series]]
- [[intent-vs-outcome-valuation]] · [[counterfactual-simulation]]
- [[split-half-reliability]] · [[predictive-validity]] · [[selection-bias]]
- [[expected-threat]] · [[vaep]] · [[scoutgpt]]
- [[action-valuation-frameworks-compared]]
- [[football-performance-time-series|Source Summary]]
