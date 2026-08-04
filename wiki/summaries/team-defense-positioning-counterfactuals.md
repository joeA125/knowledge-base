---
title: "Evaluation of Team Defense Positioning by Computing Counterfactuals using StatsBomb 360 data (Umemoto & Fujii, 2023)"
type: source_summary
tags: [summary, sports-analytics, defensive-valuation, off-ball, counterfactual, pitch-control, probability-surface, optical-tracking-data, event-stream-data, interpretability, evaluation]
sources: [raw/papers/team_defense_positioning_statsbomb.md]
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

# Evaluation of Team Defense Positioning by Computing Counterfactuals using StatsBomb 360 data

**Authors:** [[rikuhei-umemoto]] · [[keisuke-fujii]] — Graduate School of Informatics, [[nagoya-university]]
**Venue:** StatsBomb Conference 2023 (final draft)
**Data:** 290 Premier League games in 2021-22 and 290 in 2022-23, StatsBomb event + 360 data. Analysis restricted to 180 games among the ten teams with full-season coverage.

**The vault's acquisition priority 1 across six log entries.** Carried as the work that would close individual defensive credit. The answer turns out to be *nearly* — see below.

## Two Methods

### EF-OBSO — Event and Freeze-frame based OBSO

[[obso|Spearman's OBSO]] needs positions and velocities for all 22 players. StatsBomb 360 gives a **freeze frame** of whoever was visible in broadcast at each event, with no identities and no velocities.

The fix is a restriction: **compute only for events in the attacking third.** Three stated justifications, and they are better than they first look:

1. **More players visible**, especially the defending goalkeeper — 54,359 of 92,294 attacking-third events show one, against 24,970 of 290,711 elsewhere. Since OBSO zeroes offside attackers, knowing where the keeper is matters.
2. **Movement direction becomes assumable.** Elsewhere a forward may drop into midfield; in the attacking third both sides are oriented to the same goal, so "moving toward goal" is a defensible default in the absence of velocity data.
3. **It is where the question lives.** The point is defensive positioning under threat.

Velocities are then *assumed* rather than measured, and the paper tests five settings. PPCF is computed on a 32×50 grid, with a modification: for high passes, players cannot intercept mid-flight, so control is evaluated only at the arrival point.

### DRSO — Defense Response to Scoring Opportunity

The counterfactual step, and the reason this paper was wanted.

1. Find the pitch point of maximum observed EF-OBSO — the most dangerous location for the defending team.
2. Identify the **defender nearest that point**, and the grid cell he occupies.
3. Recompute EF-OBSO with that defender moved to each of the **four vertices** of his cell.
4. The vertex minimising maximum EF-OBSO is his **optimal position**.
5. $Diff_{opt-obs}$ = optimal minus observed. Closer to zero means the defender was already well placed.

Applied to the **three defenders nearest the danger point**, averaged, then averaged again per team.

**No machine learning anywhere**, deliberately — the authors argue interpretability is what makes advice actionable for coaches. See [[interpretability]].

## Speed Verification

Five velocity assumptions tested, with RMSE against actual goal/no-goal:

| Condition | All attackers (mean ×10⁻¹) | Scorers | Non-scorers |
|---|---|---|---|
| I. All 0.0 m/s | 1.246 | 8.456 | 1.214 |
| II. Others 0.0 toward goal | 1.245 | 8.472 | **1.200** |
| III. Others 4.0 | 1.196 | 8.330 | 1.340 |
| IV. Others 4.5 | 1.190 | 8.303 | 1.365 |
| V. Others 5.0 | **1.186** | **8.276** | 1.389 |

Condition V chosen, since scorers are the harder and scarcer case. Note the **trade-off**: conditions best for scorers are worst for non-scorers, and nothing optimises both.

Two observations the paper draws:

- **Scorer RMSE is an order of magnitude worse than non-scorer RMSE.** OBSO is a product of three probabilities, so even individually high terms multiply small. Places with genuinely high EF-OBSO are rare.
- **Non-scorer RMSE ≈ 0.119–0.134 sits close to the dataset's goals-per-shot ratio of 0.1156.** A neat consistency check: the model's error on non-scorers is about the base rate it should be predicting.

## Results

**Against concedes** (ten teams, 2021-22 and 2022-23 combined): Everton best on $Diff_{opt-obs}$ (−0.0395), Leeds United worst (−0.0518). Leeds were relegated.

**But Manchester City had the fewest concedes (31) and a poor DRSO (−0.0482)**, and the authors diagnose why: City hold 60%+ possession, so opponents rarely enter their attacking third — and when they do, EF-OBSO is high because City's shape is not organised for defending. **DRSO is confounded by possession share**, and the authors propose weighting by time or event count in the attacking third as a fix.

**Season-on-season**: Arsenal, Leeds and Manchester United improved; Brentford, Chelsea, Everton, Liverpool and Manchester City worsened. Brentford and Everton conceded *fewer* despite worse DRSO — traced to goalkeeper save percentage (Brentford 64/93 → 90/108), which the method does not measure.

## Does It Close Individual Defensive Credit?

**Nearly, and the distinction matters.**

The **mechanism individuates**: DRSO computes an optimal position and a $Diff_{opt-obs}$ for *each named defender*. That is genuinely per-player, and it is what the vault predicted counterfactual positioning would deliver.

The **reported evaluation does not**. Every result in the paper averages over three defenders, then over events, then over teams. Figures 6 and 7 rank *teams*. No player-level table appears anywhere.

So the vault's long-standing claim needs splitting rather than retiring:

- **"No held framework computes per-defender credit"** — now false. DRSO does.
- **"No held framework reports per-defender credit"** — still true.

The gap between those two is one aggregation step the authors chose not to skip. Nothing in the method prevents reporting per-player; they simply did not.

## Assessment

**Strengths.** The attacking-third restriction is a genuinely clever accommodation to incomplete data, justified on three independent grounds rather than convenience. The speed verification is a real sensitivity analysis, rare here. Deliberate avoidance of ML for interpretability is a defensible and unusual choice. And the Manchester City diagnosis is honest — the authors identify a confound that makes their own headline comparison misleading.

**Weaknesses.**

- **Four candidate positions only** — the vertices of the defender's current grid cell. The authors name this and propose sampling within a speed-reachable circle instead.
- **No check that the optimal position was reachable.** A defender cannot teleport to a vertex.
- **Velocities are assumed, not measured**, and the best assumption differs by target.
- **Possession-share confound**, acknowledged.
- **Goalkeeping excluded**, which is exactly what explains two of the paper's own anomalies.
- **No comparison against complete tracking data** — the authors flag this as the first limitation.
- **Ten teams, one league.**

## PPCF Parameters: A Propagated Error

The paper sets $\sigma = 0.45$ and $\lambda = 4.3$ (12.9 for goalkeepers) "following the previous study [6]", where [6] is [[beyond-expected-goals|Spearman (2018)]].

**Spearman (2018) fits $s = 0.54$ and $\lambda = 3.99$.** The values used here match neither, and the vault flagged this discrepancy when the primary Spearman source was acquired — noting the recorded values probably came from the 2017 paper rather than the 2018 one.

Two Fujii-group papers now use 0.45/4.3 while citing Spearman 2018. That is a **citation error propagating through a research line**, and the vault caught it only because it holds the primary source. See [[obso]].

The goalkeeper multiplier ($\lambda = 12.9$, three times the outfield rate) is new here and is *not* Spearman's $\kappa = 1.72$ defensive advantage — a different asymmetry, differently motivated.

## See Also

- [[drso]] · [[obso]] · [[defensive-valuation]] · [[counterfactual-baseline]] · [[pitch-control]]
- [[gvdep]] · [[vdep]] · [[c-obso]] · [[off-ball-value]] · [[interpretability]]
- [[rikuhei-umemoto]] · [[keisuke-fujii]] · [[william-spearman]] · [[nagoya-university]]
- [[beyond-expected-goals|Spearman Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[action-valuation-frameworks-compared]]
