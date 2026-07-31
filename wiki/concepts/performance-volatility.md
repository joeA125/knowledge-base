---
title: "Performance Volatility"
type: concept
tags: [sports-analytics, player-evaluation, volatility, time-series, statistics, reliability, evaluation, recruitment]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.8
provenance:
  extracted: 60%
  inferred: 22%
  generated: 13%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Performance Volatility

Volatility measures how much a player's match-to-match output deviates from their own established level. It is a property of an individual's series over time, distinct from how players differ from each other, and it captures something coaches routinely talk about but metrics rarely quantify: **can you rely on this player to turn up?**

## Motivation

Two players with identical season averages are not interchangeable. One delivers close to their average every week; the other alternates between decisive and invisible. For a side chasing a title, the consistent player may be worth more despite the identical mean.

## The Three Measures

[[football-performance-time-series|Mendes-Neves et al.]] define volatility as the standard deviation of deviations from the player's own long-term level, where $r_G$, $r_{ST}$ and $r_{LT}$ are the per-game, short-term and long-term [[player-rating-time-series|rating series]]:

$$\text{Game-to-Game Volatility} = \sigma(r_G - r_{LT})$$

$$\text{Negative Game Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_G - r_{LT}$$

$$\text{Negative Short-Term Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_{ST} - r_{LT}$$

Two design decisions carry the weight.

### Downside-only variance

The second and third measures **zero out positive deviations**, penalising only underperformance. This encodes a real asymmetry: a player who occasionally produces a masterclass is not "unreliable" in any sense a manager minds.

This mirrors downside deviation and the Sortino ratio in finance, where upside volatility is likewise not treated as risk.^[imported: the finance parallel is background knowledge; the source uses adjacent vocabulary but does not name these measures]

### Two timescales of failure

The third measure uses the **short-term average** rather than single games, separating a bad match from a bad month. Negative game volatility catches occasional anonymous afternoons; negative short-term volatility catches the player who disappears for six weeks — a different and arguably more damaging failure mode.

## The Rating-Level Correction

Raw volatility is **confounded with quality**. Higher-rated players generate more value, so their deviations are larger in absolute terms — a naive ranking would re-rank players by ability with the sign flipped.

The fix: regress volatility on rating across all players and **take the residual**, asking whether a player is more or less consistent *than players of similar quality*.

> A subtlety the source does not address: it assumes the volatility–rating relationship is **linear**.^[generated: this objection is raised here, not by the source] If dispersion scales roughly proportionally with level — as it often does for non-negative quantities — a multiplicative correction, or modelling volatility on a log scale, would be better specified. With ratings spanning a modest range this may not matter much in practice.

## Relation to Split-Half Reliability

Both concepts measure consistency, and they are easy to confuse.

| | [[split-half-reliability]] | Performance volatility |
|---|---|---|
| Unit | The **metric** | The **player** |
| Question | Does this rating replicate on another sample? | Does this player replicate week to week? |
| Computed | Correlation across players, between halves | Standard deviation within one player's series |
| High value means | The metric is trustworthy | The player is *in*consistent |
| Treats variation as | Noise | Signal about the player |

### They are the same variance component

**This is not two phenomena.**^[generated: neither source draws this equivalence; it is constructed from the two together] Writing a per-game rating as $r_{ig} = \theta_i + \varepsilon_{ig}$, the split-half correlation is

$$\rho = \frac{\sigma^2_\theta}{\sigma^2_\theta + 2\sigma^2_\varepsilon / n}$$

so a low $\rho$ says $\sigma_\varepsilon$ is large relative to between-player spread. Game-to-game volatility, $\sigma(r_G - r_{LT})$, **estimates that same $\sigma_\varepsilon$.**

One variance, two names. The reliability framing calls it noise because it assumes $\theta_i$ is fixed; this page's framing assumes $\theta_i(t)$ moves. See the full decomposition on [[split-half-reliability]].

Two consequences follow, and both bear directly on how this page should be used:

**Volatility and reliability are not independent evidence.** They should not be cited as two findings supporting one conclusion about a metric or a player.

**The question is decidable, not merely open.** The earlier framing here — "both may be partly right, and untangling them is an open problem" — understated what can be done. The real question is narrower: **is $\varepsilon$ exchangeable noise, or a stable property of the player?**

Two tests answer it, and neither needs new data:

1. **Split-half reliability of the volatility metric itself.** If volatility replicates across halves of a season, it is a real trait and the noise reading is wrong. Runnable today.
2. **The aggregation-ratio test** on [[split-half-reliability]], comparing dispersion at game and 10-game scales against the $\approx 0.28$ ratio i.i.d. noise predicts. Currently blocked by a data-fidelity problem in the source — see below.

See [[within-season-variation-noise-or-signal]], which develops both and sets out how to read them jointly. A player can be reliably streaky without their streaks being *forecastable*, and the two tests separate those cases — which determines whether volatility belongs in squad **risk** assessment only, or in match-by-match selection too.

> ⚠️ **Data fidelity.** The league-median volatility figures needed for test 2 do not survive in the vault's copy of the source: the Fig. 4 table is a fabricated arithmetic ramp rather than measurement. See the Data Fidelity section of [[football-performance-time-series]]. Test 1 is unaffected.

## Practical Use

- **Squad construction** — pair high-volatility, high-ceiling players with consistent ones, rather than optimising every slot for mean output.
- **Risk-adjusted valuation** — treat the consistent player as the safer asset at equal expected contribution.
- **Contract and transfer decisions** — a high negative short-term volatility profile suggests a player prone to extended slumps, a distinct risk from generally lower output.

All three assume volatility is a player property. **If test 1 comes back negative, none of them is supportable** — which is worth knowing before building a recruitment process on them.

**Explicitly not done, and flagged by the authors as future work:** conditioning volatility on **opposition difficulty**, separating players who raise their level against strong sides from those who accumulate against weak ones. [[league-strength-rating|Mean opponent rating]] now supplies the missing ingredient, and nobody has run it.

## Limitations

- **Confounded with rotation and role.** A player used in varied positions or as an impact substitute looks volatile for unrelated reasons.
- **Inherits the base metric's noise.** Built on VAEP, volatility partly measures the variance of rare goal events rather than the player's consistency — the confound above, unresolved.
- **Ignores context entirely.** Opposition, home/away, match state and teammate availability are all absorbed.
- **Requires a long series**, so it is unavailable for exactly the young and newly-signed players whose reliability is least known.

## See Also

- [[within-season-variation-noise-or-signal]] — the open question, with both tests
- [[split-half-reliability]] · [[player-rating-time-series]] · [[player-development-curve]] · [[predictive-validity]]
- [[vaep]] · [[action-valuation]] · [[recruitment]] · [[league-strength-rating]]
- [[football-performance-time-series|Source Summary]]
