---
title: "Is within-season variation noise or signal?"
type: question
tags: [reliability, volatility, evaluation, predictive-validity, player-evaluation, statistics, time-series, sports-analytics, needs-review]
sources: [raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/football-performance-time-series.md]
confidence: 0.75
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Is within-season variation noise or signal?

**Status:** Open, but **analytically sharper than the vault has recorded** — the two positions turn out to be measuring the same quantity, which makes the question decidable by a single test.

Two held pages take opposite views of the same numbers.

**[[split-half-reliability]]** splits a season's matches at random, computes player ratings on each half, and correlates. [[vaep]] scores $\rho = 0.25$ against [[expected-threat|xT]]'s $0.89$, and VAEP is marked down accordingly. **Within-season variation is treated as measurement error.**

**[[performance-volatility]]** takes the same match-to-match variation and treats it as a **property of the player** — consistency against streakiness — proposing downside deviation, residualised against the player's own trend, as a squad-building input.

Both cannot be wholly right about the same variance.

## They Are Measuring the Same Quantity

Model a player's per-match value as $X_i = \mu_p + \varepsilon_i$, with $\mu_p$ the player's underlying level and $\varepsilon_i$ match-to-match departure from it. Each random half of $n$ matches gives a mean of roughly $\mu_p + \varepsilon/\sqrt{n/2}$, so the across-player correlation is approximately

$$\rho \approx \frac{\sigma^2_{\text{between}}}{\sigma^2_{\text{between}} + 2\sigma^2_{\varepsilon}/n}$$

**Split-half reliability is low precisely when $\sigma_\varepsilon$ is high.** And $\sigma_\varepsilon$ — the spread of departures from a player's own level — is what volatility metrics measure.

Two consequences follow, and neither is stated in the vault.

**The two metrics are near-deterministic functions of each other**, given between-player spread and sample size. Reporting both as independent findings overstates the evidence.

**Random splitting matters.** Because each half samples across the whole season, a *slow* form trend affects both halves alike and does not depress $\rho$. What depresses $\rho$ is high-frequency, match-to-match variation. So split-half reliability specifically penalises **erratic** variation, not development or form cycles — which is exactly the component volatility metrics isolate by residualising against trend.

So this is not a dispute about different quantities. It is a dispute about **whether $\varepsilon$ is exchangeable noise or a stable player property.**

## The Test That Settles It

If $\varepsilon$ is noise, it is exchangeable, and a player's volatility in one half of a season tells you nothing about the other half. If $\varepsilon$ is a property, volatility persists.

**Compute the split-half reliability of the volatility metric itself.**

- **High** — volatility is a real, measurable player characteristic. The "noise" reading is wrong, and [[vaep]]'s low $\rho$ is partly measuring genuine player inconsistency rather than metric failure. VAEP would be partially exonerated.
- **Low** — volatility is not a stable property, the "signal" reading is wrong, and volatility metrics should not inform recruitment.

The test is cheap, needs no new data, and uses machinery both sides already have. That nobody has run it is the notable part: **each side would have to apply the other's tool to its own claim.**

## A Second, Complementary Test

The [[action-valuation-frameworks-compared|synthesis]] has carried a different formulation for several entries: does short-term deviation from a player's long-term level **predict next-match contribution** beyond the long-term average alone?

That asks whether $\varepsilon$ is *forecastable*, which is stronger than whether it is *stable*. A player could be reliably streaky without their streaks being predictable in advance.

Run in order, the two are informative jointly:

| Volatility replicates? | $\varepsilon$ forecasts? | Reading |
|---|---|---|
| No | No | Pure noise. Volatility metrics should be dropped |
| **Yes** | No | A real trait, not exploitable. Useful for squad *risk*, not selection |
| Yes | **Yes** | Form is real and predictable — the strongest case, and the most surprising |
| No | Yes | Incoherent; would indicate a bug |

The second row is the most likely and is the useful answer: it would justify [[performance-volatility]] for portfolio construction while denying it any role in match-by-match prediction.

## Why This Matters Beyond the Two Pages

The [[recruitment]] recommendation currently rests on the noise reading. xT is preferred over VAEP for season-long decisions *because* it replicates better. If VAEP's low $\rho$ is substantially genuine player inconsistency, that recommendation is measuring the wrong thing — VAEP would be reporting real variation and being penalised for it.

It also bears on every reliability figure in the vault. **No off-ball or defensive metric has a reported $\rho$**, and if $\rho$ conflates metric noise with player inconsistency, those missing numbers would be harder to interpret than their absence suggests.

## Why Nobody Has Done It

The two positions come from different papers with different purposes — [[on-ball-actions-football-xt-vs-vaep|Van Roy et al.]] are critiquing a metric, [[football-performance-time-series|Mendes-Neves et al.]] are building one — and neither cites the other's framing of the variance.

The contradiction is only visible from holding both. Like [[observed-versus-optimal-decisions]], **no individual author owns the claim**, so no individual author has reason to resolve it.

## See Also

- [[split-half-reliability]] · [[performance-volatility]] · [[player-rating-time-series]] · [[predictive-validity]]
- [[vaep]] · [[expected-threat]] · [[recruitment]] · [[player-development-curve]]
- [[observed-versus-optimal-decisions]] · [[action-valuation-frameworks-compared]]
- [[on-ball-actions-football-xt-vs-vaep|xT/VAEP Summary]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
