---
title: "Performance Volatility"
type: concept
tags: [sports-analytics, player-evaluation, volatility, time-series, statistics, reliability, evaluation, recruitment]
sources: [raw/papers/football-performance-time-series.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.8
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Performance Volatility

Volatility measures how much a player's match-to-match output deviates from their own established level. It is a property of an individual's series over time, distinct from how players differ from each other, and it captures something coaches routinely talk about but metrics rarely quantify: **can you rely on this player to turn up?**

## Motivation

Two players with identical season averages are not interchangeable. One delivers close to their average every week; the other alternates between decisive and invisible. For a side chasing a title — where a single flat performance can cost a season — the consistent player may be worth more despite the identical mean.

## The Three Measures

[[football-performance-time-series|Mendes-Neves et al.]] define volatility as the standard deviation of deviations from the player's own long-term level, where $r_G$, $r_{ST}$ and $r_{LT}$ are the per-game, short-term and long-term [[player-rating-time-series|rating series]]:

$$\text{Game-to-Game Volatility} = \sigma(r_G - r_{LT})$$

$$\text{Negative Game Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_G - r_{LT}$$

$$\text{Negative Short-Term Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_{ST} - r_{LT}$$

Two design decisions carry all the weight.

### Downside-only variance

The second and third measures **zero out positive deviations**, penalising only underperformance. This encodes a real asymmetry in how consistency is valued: a player who occasionally produces a masterclass is not "unreliable" in any sense a manager minds. Only falling short of your own standard counts against you.

This mirrors downside deviation and the Sortino ratio in finance, where upside volatility is likewise not treated as risk. The analogy is close and the borrowing appears deliberate given the paper's vocabulary.

### Two timescales of failure

The third measure differs from the second by using the **short-term average** rather than single games. This separates a bad match from a bad month. Negative game volatility catches the player who has occasional anonymous afternoons; negative short-term volatility catches the player who disappears for six weeks at a time — a different and arguably more damaging failure mode.

## The Rating-Level Correction

Raw volatility is **confounded with quality**. Higher-rated players generate more value, so their deviations are larger in absolute terms — a naive volatility ranking would essentially re-rank players by ability with the sign flipped.

The fix: regress volatility on rating across all players and **take the residual**. This asks whether a player is more or less consistent *than players of similar quality*, which is the question actually of interest.

This is a small step with wide applicability. Any dispersion measure computed over subjects with different levels needs it, and it is a common source of spurious findings when omitted.

> A remaining subtlety the source does not address: it assumes the volatility–rating relationship is **linear**. If dispersion scales roughly proportionally with level — as it often does for non-negative quantities — a multiplicative correction, or modelling volatility on a log scale, would be better specified. With ratings spanning a modest range this may not matter much in practice.

## Relation to Split-Half Reliability

Both concepts measure consistency, and they are easy to confuse.

| | [[split-half-reliability]] | Performance volatility |
|---|---|---|
| Unit | The **metric** | The **player** |
| Question | Does this rating replicate on another sample? | Does this player replicate week to week? |
| Computed | Correlation across players, between halves | Standard deviation within one player's series |
| High value means | The metric is trustworthy | The player is *in*consistent |
| Treats variation as | Noise | Signal about the player |

The relationship is genuinely awkward. High player volatility **causes** low metric reliability — if every player's output swings wildly, no rating computed from a sample will replicate. So the same underlying variance is a *finding* under one framing and a *defect* under the other.

Distinguishing them requires separating variance in the player from variance in the measurement, which the vault's sources do not attempt. [[split-half-reliability|Van Roy et al.]] attribute VAEP's low reliability primarily to goals being rare and heavily weighted — a **measurement** property. Mendes-Neves et al. interpret the same kind of dispersion as a **player** property. Both may be partly right, and untangling them is an open problem.

## Practical Use

- **Squad construction** — pair high-volatility, high-ceiling players with consistent ones, rather than optimising every slot for mean output.
- **Risk-adjusted valuation** — treat the consistent player as the safer asset at equal expected contribution.
- **Contract and transfer decisions** — a high negative short-term volatility profile suggests a player prone to extended slumps, a distinct risk from generally lower output.

**Explicitly not done, and flagged by the authors as future work:** conditioning volatility on **opposition difficulty**. This would separate players who raise their level against strong sides from those who accumulate value against weak ones — likely the most valuable version of the metric, and untested.

## Limitations

- **Confounded with rotation and role.** A player used in varied positions or as an impact substitute will look volatile for reasons unrelated to reliability.
- **Inherits the base metric's noise.** Built on VAEP, volatility partly measures the variance of rare goal events rather than the player's actual consistency — the confound above, unresolved.
- **Ignores context entirely.** Opposition, home/away, match state, and teammate availability are all absorbed into "volatility".
- **Requires a long series**, so it is unavailable for exactly the young and newly-signed players whose reliability is least known.

## See Also

- [[player-rating-time-series]]
- [[split-half-reliability]]
- [[player-development-curve]]
- [[predictive-validity]]
- [[vaep]]
- [[action-valuation]]
- [[football-performance-time-series|Source Summary]]
