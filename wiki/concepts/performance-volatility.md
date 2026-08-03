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

Volatility measures how much a player's match-to-match output deviates from their own established level. It captures something coaches routinely talk about but metrics rarely quantify: **can you rely on this player to turn up?**

## Motivation

Two players with identical season averages are not interchangeable. One delivers close to their average every week; the other alternates between decisive and invisible. For a side chasing a title, the consistent player may be worth more despite the identical mean.

## The Three Measures

[[football-performance-time-series|Mendes-Neves et al.]] define volatility as the standard deviation of deviations from the player's own long-term level, where $r_G$, $r_{ST}$ and $r_{LT}$ are the per-game, short-term and long-term [[player-rating-time-series|rating series]]:

$$\text{Game-to-Game Volatility} = \sigma(r_G - r_{LT})$$

$$\text{Negative Game Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_G - r_{LT}$$

$$\text{Negative Short-Term Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_{ST} - r_{LT}$$

### Downside-only variance

The second and third **zero out positive deviations**, penalising only underperformance. A player who occasionally produces a masterclass is not "unreliable" in any sense a manager minds.

This mirrors downside deviation and the Sortino ratio in finance.^[imported: the finance parallel is background knowledge; the source uses adjacent vocabulary but does not name these measures]

### Two timescales of failure

The third uses the **short-term average** rather than single games, separating a bad match from a bad month — a different and arguably more damaging failure mode.

## The Rating-Level Correction

Raw volatility is **confounded with quality**. Higher-rated players generate more value, so their deviations are larger in absolute terms.

The fix: regress volatility on rating across all players and **take the residual**, asking whether a player is more or less consistent *than players of similar quality*.

> A subtlety the source does not address: it assumes the volatility–rating relationship is **linear**.^[generated: this objection is raised here, not by the source. rests-on: source:mendes-neves-residual-correction] If dispersion scales proportionally with level — as it often does for non-negative quantities — a multiplicative correction, or modelling on a log scale, would be better specified.

## Relation to Split-Half Reliability

| | [[split-half-reliability]] | Performance volatility |
|---|---|---|
| Unit | The **metric** | The **player** |
| Question | Does this rating replicate on another sample? | Does this player replicate week to week? |
| High value means | The metric is trustworthy | The player is *in*consistent |
| Treats variation as | Noise | Signal about the player |

### They are the same variance component

**`reliability-volatility-identity`** — split-half reliability and performance volatility measure the same variance component, with opposite interpretations.^[generated: declared on [[split-half-reliability]], where the decomposition and both tests are set out. Its load-bearing premise is *imported* standard psychometrics, so nothing in the vault can check it. rests-on: claim:reliability-volatility-identity]

Writing a per-game rating as $r_{ig} = \theta_i + \varepsilon_{ig}$, the split-half correlation is

$$\rho = \frac{\sigma^2_\theta}{\sigma^2_\theta + 2\sigma^2_\varepsilon / n}$$

so a low $\rho$ says $\sigma_\varepsilon$ is large relative to between-player spread. Game-to-game volatility, $\sigma(r_G - r_{LT})$, **estimates that same $\sigma_\varepsilon$.**

One variance, two names. The reliability framing calls it noise because it assumes $\theta_i$ is fixed; this page's framing assumes $\theta_i(t)$ moves.

Two consequences:

**Volatility and reliability are not independent evidence.** They should not be cited as two findings supporting one conclusion.

**The question is decidable, not merely open.** An earlier framing here — "both may be partly right, and untangling them is an open problem" — understated what can be done. The real question is narrower: **is $\varepsilon$ exchangeable noise, or a stable property of the player?**

Two tests, neither needing new data:

1. **Split-half reliability of the volatility metric itself.** If volatility replicates across halves of a season, it is a real trait. Runnable today.
2. **The aggregation-ratio test** on [[split-half-reliability]], comparing dispersion at game and 10-game scales against the $\approx 0.28$ ratio i.i.d. noise predicts. Currently blocked by a data-fidelity problem in the source.

See [[within-season-variation-noise-or-signal]], which develops both and sets out how to read them jointly. A player can be reliably streaky without their streaks being *forecastable*, and the two tests separate those cases.

> ⚠️ **Data fidelity.** The league-median volatility figures needed for test 2 do not survive in the vault's copy of the source: the Fig. 4 table is a fabricated arithmetic ramp rather than measurement. See the Data Fidelity section of [[football-performance-time-series]]. Test 1 is unaffected.

## Practical Use

- **Squad construction** — pair high-volatility, high-ceiling players with consistent ones, rather than optimising every slot for mean output.
- **Risk-adjusted valuation** — treat the consistent player as the safer asset at equal expected contribution.
- **Contract and transfer decisions** — a high negative short-term volatility profile suggests a player prone to extended slumps.

All three assume volatility is a player property. **If test 1 comes back negative, none of them is supportable** — worth knowing before building a recruitment process on them.

**Explicitly not done, flagged by the authors as future work:** conditioning volatility on **opposition difficulty**, separating players who raise their level against strong sides from those who accumulate against weak ones. [[league-strength-rating|Mean opponent rating]] now supplies the missing ingredient.

## Limitations

- **Confounded with rotation and role.** A player used in varied positions or as an impact substitute looks volatile for unrelated reasons.
- **Inherits the base metric's noise.** Built on VAEP, volatility partly measures the variance of rare goal events.
- **Ignores context entirely.** Opposition, home/away, match state and teammate availability are all absorbed.
- **Requires a long series**, so it is unavailable for exactly the young and newly-signed players whose reliability is least known.

## See Also

- [[within-season-variation-noise-or-signal]] — the open question, with both tests
- [[split-half-reliability]] · [[player-rating-time-series]] · [[player-development-curve]] · [[predictive-validity]]
- [[vaep]] · [[action-valuation]] · [[recruitment]] · [[league-strength-rating]]
- [[football-performance-time-series|Source Summary]]
