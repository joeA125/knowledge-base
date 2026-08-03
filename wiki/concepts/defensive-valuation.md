---
title: "Defensive Valuation"
type: concept
tags: [defensive-valuation, sports-analytics, action-valuation, off-ball, player-evaluation, evaluation, optical-tracking-data, proxy-target, counterfactual]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 25%
  generated: 10%
  imported: 0%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Defensive Valuation

Quantifying the contribution of preventing goals rather than creating them. Long the least-served problem in this vault's football coverage.

## Why It Is Structurally Harder

**The target is a non-event.** Defensive success is the *absence* of an outcome, and absences are hard to attribute.

**The base rate is punishing.** A conceding classifier sees ~0.2% positives. See [[rare-event-proxy-targets]].

**Credit is diffuse.** A defensive success involves the presser, the cover, the marker, and the shape they collectively hold — most of whom never touch the ball.

**The data does not record it.** [[event-stream-data|Event data]] logs actions; pressing, screening and holding a line generate none. Only [[optical-tracking-data|tracking]] fixes this — though [[gvdep|GVDEP]] shows partial observation goes further than expected.

## The Symptom Across the Vault

Van Dijk is **81st by [[vaep]], 142nd by [[expected-threat|xT]]**. Two results complicate the reading that this is purely definitional:

- He tops **both** of [[duel-skill-rating|Shelopugin's duel-rating tables]] — the information exists in ordinary event data, unmodelled.
- [[vaep]]'s conceding classifier scores F1 = 0.000 on 45 matches. ⚠️ Near-guaranteed for any calibrated model at a 0.23% base rate, and VAEP never thresholds. [[gvdep|GVDEP]] reports 0.08–0.15 on a different dataset, so the zero is not a fixed property of the model. See [[vaep-conceding-classifier]].

This is the third of the four causes in `offensive-bias-four-causes`^[generated: declared on [[action-valuation]]. The fourth cause rests on the F1 finding flagged above and is the least secure. rests-on: claim:offensive-bias-four-causes].

## Five Approaches

| Approach | Idea | Example | Level | Held? |
|---|---|---|---|---|
| Negative half of an attacking model | Value defence as reduced $P(\text{concede})$ | [[vaep]] | Player | Yes |
| Contested-event rating | Rate who wins physical contests | [[duel-skill-rating]] | Player | Yes |
| Frequent proxy prediction | Predict recovery / being attacked | [[vdep]], [[gvdep]] | **Team** | Yes |
| **Counterfactual positioning** | What would a different position have prevented? | Umemoto & Fujii (**2023**) | **Player** | **No** |
| Spatial suppression | Model the space a defence denies | [[pitch-control]] | Team, implicit | Yes |

## The Proxy-Target Line

Two held frameworks, the second fixing the first.

**[[vdep|VDEP]]** (Toda et al., 2022) establishes the approach: predict **ball recovery** and **being attacked** instead of conceding, raising F1 from 0.000 to 0.522 and 0.484. [[shap]] confirms off-ball features carry the signal.

**[[gvdep|GVDEP]]** (Umemoto, Tsutsui & Fujii, 2022) fixes three stated limitations:

- VDEP's weight $C$ came from the **frequency ratio** of the two events. GVDEP replaces it with **[[vaep|VAEP]] evaluated at the moments those events occur**, putting both terms on a score scale. See [[model-selection]].
- VDEP assumed **all 22 players observed**. GVDEP uses broadcast frames and measures what the shortfall costs: ball-gain prediction saturates at **three or four players**; scores, concedes and being-attacked gain nothing from player positions at all.
- VDEP used one domestic men's league. GVDEP covers **men's Euro 2020 and women's Euro 2022**.

**The gain/attacked trade-off replicates** across both ($r = -0.757$ in Euro 2020; same direction in the J-League) — one of the few genuine replications in this literature.

GVDEP's own limitation is that it correlates **0.993 with its attacked term alone**, so the metric is nearly a monotone function of one component. Its authors flag this.

## Individual Credit Remains Open

> **Superseded, 2026-07-27.** This page previously implied the Umemoto line had closed the individual-defender gap. **It has not.** [[gvdep|GVDEP]] — now held — is **team-level**, exactly like its predecessor.

The vault **conflated two papers by the same author**:

| Work | What it does | Held? |
|---|---|---|
| Umemoto, Tsutsui & Fujii (**2022**), arXiv:2212.00021 | [[gvdep\|GVDEP]] — team-level, fixes VDEP's weighting and observation assumptions | **Yes** |
| Umemoto & Fujii (**2023**), StatsBomb Conference | **Counterfactual positioning — individual defenders** | **No** |

Acquiring the first did nothing for the gap the second would close. That failure mode is worth naming: **a vault can appear to have closed a gap because it acquired a paper by the right author.** Bibliographic precision on cited-not-held work matters more than it looks.

Individual defensive credit is therefore **still unaddressed in `raw/`**, and the 2023 paper remains the vault's top acquisition target.

### Why the counterfactual route is the plausible one

**`counterfactual-individuates`** — the individuating ingredient is the counterfactual, not the data.^[generated: declared on [[counterfactual-baseline]], where a Shapley-style objection demotes it from law to tendency. rests-on: claim:counterfactual-individuates]

[[vdep]], [[gvdep]] and [[c-obso]] use comparable tracking data. The first two produce one number per configuration; C-OBSO intervenes on one *named* player and produces his number. On that reading, counterfactual positioning would individuate defensive credit for the same structural reason — but the inference rests on a generated premise that has itself been demoted, and should not be treated as established.

As described by Fujii in an overview article, the 2023 method identifies the highest-[[obso|OBSO]] location, selects the closest defender, and searches which cell he could have occupied to most reduce OBSO. Both Fujii counterfactual lines build on [[obso|OBSO]] rather than event classification.

### Cost

Fujii reports the trajectory method needed **enormous computation to evaluate all 22 players**. [[c-obso]] confirms it: only three of 22 are predicted. GVDEP's n_nearest result offers a possible reprieve — if three or four players suffice for the prediction that matters, full-squad evaluation may be unnecessary rather than merely expensive.

## What Remains Open

- **Individual defensive credit is unheld**, as above.
- **No cross-framework comparison** with anything outside this line.^[generated: an instance of `no-cross-framework-benchmarking`, declared on the synthesis. Weakened: GVDEP is compared directly against VDEP on identical data, though by the same group. rests-on: claim:no-cross-framework-benchmarking]
- **Errors of omission** — a defender who fails to press.
- **Reliability unreported** for every defensive metric here.^[generated: an instance of `no-reliability-for-off-ball-metrics`, declared on the synthesis. rests-on: claim:no-reliability-for-off-ball-metrics]
- **Goalkeeping excluded** by GVDEP, which misprices teams that defend by shot-stopping — Belgium and the Czech Republic score poorly despite few concedes.

## Beyond Football

The structure recurs wherever the valuable outcome is prevented: fraud not committed, outages avoided, infections not transmitted.^[imported: background knowledge, not from any held source] Same three problems — the target is an absence, the base rate is tiny, credit is shared across a system.

## See Also

- [[vdep]] · [[gvdep]] · [[c-obso]] · [[counterfactual-baseline]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[vaep-conceding-classifier]] — the open question on the F1 evidence
- [[action-valuation]] · [[vaep]] · [[duel-skill-rating]] · [[off-ball-value]] · [[obso]] · [[pitch-control]] · [[model-selection]]
- [[rikuhei-umemoto]] · [[keisuke-fujii]] · [[kosuke-toda]] · [[shap]] · [[optical-tracking-data]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
