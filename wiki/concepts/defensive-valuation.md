---
title: "Defensive Valuation"
type: concept
tags: [defensive-valuation, sports-analytics, action-valuation, off-ball, player-evaluation, evaluation, optical-tracking-data, proxy-target, counterfactual]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 25%
  generated: 10%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Defensive Valuation

Quantifying the contribution of preventing goals rather than creating them. Long the least-served problem in this vault's football coverage — and now, with four held frameworks, one of the better-served.

## Why It Is Structurally Harder

**The target is a non-event.** Defensive success is the *absence* of an outcome, and absences are hard to attribute.

**The base rate is punishing.** A conceding classifier sees ~0.2% positives. See [[rare-event-proxy-targets]].

**Credit is diffuse.** A defensive success involves the presser, the cover, the marker, and the shape they collectively hold — most of whom never touch the ball.

**The data does not record it.** [[event-stream-data|Event data]] logs actions; pressing, screening and holding a line generate none. Only [[optical-tracking-data|tracking]] fixes this — though [[gvdep|GVDEP]] and [[drso|DRSO]] both show partial observation goes further than expected.

## The Symptom

Van Dijk is **81st by [[vaep]], 142nd by [[expected-threat|xT]]**, while topping **both** of [[duel-skill-rating|Shelopugin's duel-rating tables]] — the information exists in ordinary event data, unmodelled.

[[vaep]]'s conceding classifier scores F1 = 0.000 on 45 matches. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds; [[gvdep|GVDEP]] reports 0.08–0.15 on comparable data. See [[vaep-conceding-classifier]].

This is the third of the four causes in `offensive-bias-four-causes`^[generated: declared on [[action-valuation]]. rests-on: claim:offensive-bias-four-causes].

## Five Approaches, Four Now Held

| Approach | Idea | Example | Unit reported | Held? |
|---|---|---|---|---|
| Negative half of an attacking model | Value defence as reduced $P(\text{concede})$ | [[vaep]] | Player | Yes |
| Contested-event rating | Rate who wins physical contests | [[duel-skill-rating]] | Player | Yes |
| Frequent proxy prediction | Predict recovery / being attacked | [[vdep]], [[gvdep]] | **Team** | Yes |
| **Counterfactual positioning** | Where should this defender have stood? | **[[drso\|DRSO]]** | **Team** (mechanism is per-player) | **Yes** |
| Spatial suppression | Model the space a defence denies | [[pitch-control]] | Team, implicit | Yes |

## The Fujii Line, Complete

Three papers, each fixing what the previous one named:

**[[vdep|VDEP]]** (Toda et al., 2022) establishes proxy-target defensive valuation: predict **ball recovery** and **being attacked** instead of conceding, raising F1 from 0.000 to 0.522 and 0.484. Team-level, full-tracking, one league.

**[[gvdep|GVDEP]]** (Umemoto, Tsutsui & Fujii, 2022) fixes the arbitrary weight $C$ with [[vaep|VAEP]]-derived score-scaled weights, works from broadcast frames, and covers men's and women's tournaments. Still team-level. Its **n_nearest sweep** shows ball-gain prediction saturates at three or four players.

**[[drso|DRSO]]** (Umemoto & Fujii, 2023) asks a different question entirely — not *how well did the defence do* but **where should each defender have stood**. Compute EF-OBSO, find the most dangerous point, move the nearest defender to each vertex of his grid cell, and take the position minimising danger.

**The gain/attacked trade-off replicates** across VDEP and GVDEP ($r = -0.757$ in Euro 2020, same direction in the J-League) — one of the few genuine replications in this literature.

## Individual Credit: Nearly Closed

> **Superseded, 2026-07-27.** This page has carried "individual defensive credit is unaddressed in `raw/`" since the VDEP ingest, and named Umemoto & Fujii (2023) as the work that would close it. **That paper is now held.** The claim needs splitting rather than retiring.

| Claim | Status |
|---|---|
| No held framework **computes** per-defender defensive credit | **False.** [[drso\|DRSO]] computes $Diff_{opt-obs}$ for each named defender |
| No held framework **reports** per-defender defensive credit | **Still true.** Every DRSO result averages three defenders, then events, then teams |

The gap is now **one aggregation step**, not a missing method. Figures 6 and 7 of that paper rank teams; no player-level table appears. Nothing in the method prevents player-level reporting — the authors simply did not do it.

That is a materially different open question from the one the vault has been carrying. It is no longer "can this be done" but "has anyone published it".

### What this does to `counterfactual-individuates`

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]], where a Shapley-style objection demotes it from law to tendency. **Supported by DRSO**: intervening on one named defender does produce his own number, from the same collective spatial data that leaves VDEP and GVDEP stuck at team level. rests-on: claim:counterfactual-individuates]

[[vdep]], [[gvdep]] and [[drso]] use comparable data from the same research group. The first two produce one number per configuration; the third intervenes on a named defender and gets his number. That is the predicted mechanism behaving as predicted — though it remains a claim about these four instances, not a proven necessity.

## What Remains Open

- **Per-player defensive results are unpublished**, as above — the narrowest the gap has been.
- **Reachability is unchecked.** DRSO's optimal position may be somewhere the defender could not have got to.
- **Possession-share confounds DRSO.** Manchester City conceded fewest and scored poorly, because holding 60%+ possession means rare but dangerous defensive moments. Acknowledged; the proposed weighting fix is unimplemented.
- **Goalkeeping is excluded** from both GVDEP and DRSO, and explains two of DRSO's own anomalies (Brentford and Everton conceding less while scoring worse).
- **No cross-framework comparison** outside this line.^[generated: an instance of `no-cross-framework-benchmarking`, declared on the synthesis. Comparison happens within research lines and never across them. rests-on: claim:no-cross-framework-benchmarking]
- **Errors of omission** — a defender who fails to press.
- **Reliability unreported** for every defensive metric here.^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. Re-checked on the DRSO ingest, 2026-07-27: still holds. rests-on: claim:no-reliability-for-off-ball-metrics]

## Beyond Football

The structure recurs wherever the valuable outcome is prevented: fraud not committed, outages avoided, infections not transmitted.^[imported: background knowledge, not from any held source] Same three problems — the target is an absence, the base rate is tiny, credit is shared across a system.

## See Also

- [[vdep]] · [[gvdep]] · [[drso]] · [[c-obso]] · [[counterfactual-baseline]] · [[rare-event-proxy-targets]]
- [[vaep-conceding-classifier]] · [[class-imbalance-evaluation]] · [[obso]] · [[pitch-control]]
- [[action-valuation]] · [[vaep]] · [[duel-skill-rating]] · [[off-ball-value]] · [[model-selection]]
- [[rikuhei-umemoto]] · [[keisuke-fujii]] · [[kosuke-toda]] · [[shap]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
