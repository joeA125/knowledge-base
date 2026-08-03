---
title: "Location Analysis of Players in UEFA EURO 2020 and 2022 using Generalized VDEP (Umemoto, Tsutsui & Fujii, 2022)"
type: source_summary
tags: [summary, sports-analytics, defensive-valuation, off-ball, optical-tracking-data, event-stream-data, gradient-boosting, class-imbalance, proxy-target, evaluation, model-selection]
sources: [raw/papers/defensive_player_location_analysis.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Location Analysis of Players in UEFA EURO 2020 and 2022 using Generalized VDEP

**Authors:** [[rikuhei-umemoto]] · [[kazushi-tsutsui]] · [[keisuke-fujii]]
**Compiled:** December 2022. arXiv:2212.00021. Code: github.com/Rikuhei-ynwa/Generalized-VDEP
**Data:** StatsBomb open data — men's Euro 2020 (51 games), women's Euro 2022 (31 games).

The direct successor to [[football-defence-evaluation-vdep|VDEP]], addressing three limitations its authors named. Long held on the vault's acquisition list.

## What It Fixes

Toda et al. left three problems. This paper attacks each:

| VDEP limitation | GVDEP's fix |
|---|---|
| The weight $C$ between ball gain and being attacked was set by **event frequency**, encoding how *often* each happens rather than how much each *matters* | Weight both terms by **[[vaep|VAEP]] evaluated at the moments those events occur** — a score scale |
| Assumed **perfect observation of all 22 players** | Use broadcast-frame data, where most scenes show 11–18 players, and measure what the shortfall costs |
| Single domestic men's league | Men's Euro 2020 **and women's Euro 2022** |

## The Construction

$$V_{gvdep}(s_i) = w_{gains}\,\Delta P_{gains}(s_i) - w_{attacked}\,\Delta P_{attacked}(s_i)$$

with the weights averaged VAEP values at the relevant events:

$$w_{gains} = \frac{1}{|Ev_{gains}|}\sum_{j \in Ev_{gains}} \text{sign}(Team_j)\,V_{vaep}(s_j)$$

So the first term becomes the change in scoring/conceding probability *after gaining the ball*, and the second the change *after being attacked*. Four XGBoost classifiers: $P_{gains}$, $P_{attacked}$ ($k=5$), $P_{scores}$, $P_{concedes}$ ($k'=10$).

**Feature leakage is handled explicitly.** VAEP's 24 event-type features are used for the VAEP classifiers but **withheld** from the VDEP ones — knowing the action type would leak the ground truth for "did the defence gain the ball". VAEP features: 133 dims; VDEP: 109.

## The n_nearest Result

The paper's most portable finding. Sweeping the number of nearest attackers/defenders included, 0 to 11, and reporting F1 at each:

| Prediction | Behaviour as players are added |
|---|---|
| **Ball gain** | Rises from 0.16 to ~0.31, then **flat after 3–4 players** |
| Being attacked | 0.44 → 0.46, essentially flat throughout |
| Scores | 0.10 → 0.13, no gain from player data |
| Concedes | 0.15 at zero players, **falls to 0.08** with more |

Two readings, and the second is not the authors'.

**Theirs:** all 22 players are unnecessary. Ball gain needs three or four; the rest need none. So broadcast-frame data suffices, and the analysis is available to anyone with public video rather than a tracking licence.

**Also visible:** the concedes classifier gets *worse* with more player information — 0.15 down to 0.08. With 186 positive concedes in 100,328 events, extra dimensions are noise. That is the same small-data overfitting pattern [[gradient-boosting|the vault has recorded]] from Yeung & Fujii's shot data, arrived at independently.

## Results

Across the last 16 of Euro 2020:

- **The gain/attacked trade-off replicates** ($r = -0.757$, $p = 0.001$). Teams that recover more concede more territory. Same direction as VDEP's J-League finding, on a different continent and competition.
- **GVDEP correlates 0.993 with the attacked term alone** ($p = 3.2\times10^{-14}$), because $|w_{attacked}| = 0.021$ against $|w_{gains}| = 0.011$. **The authors flag this as a limitation**: the metric is nearly a monotone function of one of its two components.
- **No significant correlation with actual concedes** ($r = -0.265$, $p = 0.321$). Defended as measuring *process not outcome*, with the honest note that concedes are small integers with little variance.

Italy (champions) had the best attacked-value and a *low* gain-value — they did not try to win the ball back. England combined high values on both and conceded nothing to the round of 16. Belgium and the Czech Republic score poorly despite few concedes, which the authors attribute to goalkeeping — explicitly outside what the method measures.

## Assessment

**Strengths.** The weighting fix is principled and replaces the single most arbitrary parameter in its predecessor. The n_nearest sweep is a genuine sensitivity analysis, rare in this literature. Open data, open code, and the first vault source covering **women's football**. Feature-leakage handling is careful and explicitly reasoned.

**Weaknesses.**

- **The metric is dominated by one term** ($r = 0.993$), acknowledged.
- **Concedes F1 of 0.08–0.15** — the scoring and conceding classifiers on which the *weights* depend are themselves weak, so a principled weight is computed from an unreliable model.
- **Still team-level.** Individual defensive credit is not addressed. See below.
- $k=5$, $k'=10$ inherited from prior work without re-examination.
- No [[split-half-reliability|reliability]] figure.
- Goalkeeping explicitly excluded, which distorts exactly the teams that defend by shot-stopping.

## What This Corrects in the Vault

**Individual defensive credit remains open.** The vault has carried Umemoto & Fujii as the route to closing it. **This paper does not** — GVDEP evaluates teams, not players. The 2023 StatsBomb counterfactual-positioning work is a *different* paper and remains unheld. Pages implying otherwise are corrected.

**Three absence claims are affected** — see [[gvdep]] and the log entry for details:

- `no-sensitivity-analysis-in-held-sources` — **weakened.** The n_nearest sweep is one, and GVDEP replaces VDEP's $C$ on principle rather than asserting a new value.
- `no-held-source-propagates-tracking-error` — **weakened.** This does not propagate uncertainty, but it does measure the cost of *incomplete observation*, which is the closest thing the vault holds.
- `no-cross-framework-benchmarking` — **weakened.** GVDEP is compared against VDEP on identical data (Figure 8). Same group and its own predecessor, so not an independent benchmark, but a genuine like-for-like comparison.

## See Also

- [[gvdep]] · [[vdep]] · [[defensive-valuation]] · [[vaep]]
- [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]] · [[gradient-boosting]] · [[model-selection]]
- [[rikuhei-umemoto]] · [[kazushi-tsutsui]] · [[keisuke-fujii]] · [[kosuke-toda]] · [[nagoya-university]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[action-valuation-frameworks-compared]]
