---
title: "DRSO and EF-OBSO"
type: concept
tags: [defensive-valuation, off-ball, counterfactual, sports-analytics, pitch-control, probability-surface, optical-tracking-data, interpretability, player-evaluation]
sources: [raw/papers/team_defense_positioning_statsbomb.md, raw/papers/beyond_expected_goals.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# DRSO and EF-OBSO

[[team-defense-positioning-counterfactuals|Umemoto & Fujii's (2023)]] paired methods for evaluating **defensive positioning** by asking where each defender should have stood.

The vault's first framework that computes a **per-defender** value from collective spatial data — though, as below, it does not report one.

## EF-OBSO: Making OBSO Work on Incomplete Data

[[obso|Spearman's OBSO]] requires positions and velocities for all 22 players. StatsBomb 360 supplies a **freeze frame** of whoever was visible in broadcast at each event — no identities, no velocities, rarely all 22.

The accommodation is a restriction: **compute only for events in the attacking third**, justified on three independent grounds.

**More players visible, especially the keeper.** 54,359 of 92,294 attacking-third events show a defending goalkeeper, against 24,970 of 290,711 elsewhere. Since OBSO zeroes attackers in offside positions, the keeper's location is load-bearing.

**Direction becomes assumable.** Outside the attacking third a forward may be dropping into midfield; inside it, both teams orient to the same goal, so "moving toward goal" is defensible where velocity is unmeasurable.

**It is where the question lives.** Defensive positioning matters under threat.

One further modification: for **high passes**, players cannot intercept mid-flight, so PPCF is evaluated only at the arrival point rather than integrated along the path.

## DRSO: The Counterfactual Step

1. Find the pitch point of **maximum observed EF-OBSO** — the most dangerous location for the defence.
2. Identify the **defender nearest that point** and the grid cell he occupies.
3. Recompute EF-OBSO with that defender moved to each of the **four vertices** of his cell.
4. The vertex minimising maximum EF-OBSO is his **optimal position**.
5. $Diff_{opt-obs} = \text{EF-OBSO}_{opt} - \text{EF-OBSO}_{obs}$. Nearer zero means better placed.

Applied to the **three defenders nearest the danger point**, then averaged — per event, then per team.

This is a [[counterfactual-baseline]] whose reference is an **optimum**, not a predicted behaviour: it asks *how far from best*, where [[c-obso]] asks *how far from normal*. The two Fujii-group counterfactual methods sit on opposite ends of that axis.

## Per-Defender Mechanism, Per-Team Report

The distinction that matters for the vault's long-standing open question.

| | Status |
|---|---|
| Does the **method** compute per-defender values? | **Yes** — $Diff_{opt-obs}$ is defined for each named defender |
| Does the **paper** report per-defender values? | **No** — every result averages three defenders, then events, then teams |

Figures 6 and 7 rank teams. No player-level table appears anywhere.

So `counterfactual-individuates`^[generated: declared on [[counterfactual-baseline]]. rests-on: claim:counterfactual-individuates] is **vindicated as a claim about mechanism** — the intervention on one named defender does produce his own number, exactly as predicted — and the remaining gap is one aggregation step the authors chose not to skip. Nothing in the method prevents player-level reporting.

## No Machine Learning, Deliberately

The authors argue explicitly that ML's low interpretability limits practical application, and that a physical-model approach gives players and coaches more usable advice.

That places DRSO alongside [[obso|OBSO]] in the vault's **purely theory-based** column — see [[theory-based-modelling]]. It is also a rare case of interpretability being chosen *over* capability rather than traded against it.

## Results and the Confound

Everton best on $Diff_{opt-obs}$ (−0.0395), Leeds United worst (−0.0518); Leeds were relegated.

But **Manchester City had the fewest concedes (31) and a poor DRSO score.** The authors diagnose it themselves: City hold 60%+ possession, so opponents rarely reach their attacking third — and when they do, City's shape is not organised for it, so EF-OBSO is high.

**DRSO is confounded by possession share.** A team that defends rarely has fewer, more dangerous defensive moments. The proposed fix — weight by time or event count in the attacking third — is not implemented.

A second anomaly the method cannot see: Brentford and Everton conceded *fewer* goals while scoring *worse* on DRSO, traced to goalkeeper save percentage (Brentford 64/93 → 90/108). **Goalkeeping is outside what DRSO measures.**

## Limitations

- **Four candidate positions only**, the vertices of the current grid cell. The authors propose sampling within a speed-reachable circle instead.
- **No reachability check.** A defender cannot teleport to a vertex; whether he could have been there is untested.
- **Velocities assumed, not measured.** Five settings tested; the best for scorers is the worst for non-scorers, and nothing optimises both.
- **Possession-share confound**, acknowledged.
- **No comparison against complete tracking data** — the authors' own first-listed limitation.
- Ten teams, one league, two seasons.
- No [[split-half-reliability|reliability]] figure.

## PPCF Parameter Note

DRSO sets $\sigma = 0.45$, $\lambda = 4.3$ (12.9 for goalkeepers), citing [[beyond-expected-goals|Spearman (2018)]] — which actually fits $s = 0.54$, $\lambda = 3.99$. Two Fujii-group papers now use values that match neither, while citing that source. See [[obso]].

The goalkeeper multiplier is new and distinct from Spearman's $\kappa = 1.72$ defensive advantage: a keeper-specific control rate rather than a general defender one.

## Where It Sits

| | [[vdep]] | [[gvdep]] | **DRSO** | [[c-obso]] |
|---|---|---|---|---|
| Mechanism | Classifier on 22 positions | Same, VAEP-weighted | **Counterfactual on position** | Counterfactual on trajectory |
| Reference | — | — | **The optimum** | Predicted behaviour |
| Computes per player | No | No | **Yes** | Yes |
| Reports per player | No | No | **No** | No |
| Learned? | XGBoost | XGBoost | **No ML** | GVRNN |

## See Also

- [[obso]] · [[counterfactual-baseline]] · [[defensive-valuation]] · [[c-obso]] · [[pitch-control]]
- [[gvdep]] · [[vdep]] · [[off-ball-value]] · [[theory-based-modelling]] · [[interpretability]]
- [[rikuhei-umemoto]] · [[keisuke-fujii]] · [[william-spearman]]
- [[team-defense-positioning-counterfactuals|Source Summary]] · [[beyond-expected-goals|Spearman Summary]]
