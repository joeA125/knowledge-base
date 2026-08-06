---
title: "Are the free parameters load-bearing?"
type: question
tags: [model-selection, discounting, evaluation, sports-analytics, action-valuation, reliability, predictive-validity, space-creation, needs-review]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md, raw/papers/defensive_player_location_analysis.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/wide_open_spaces_creation_football.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 42%
  generated: 2%
  imported: 0%
  ambiguous: 1%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Are the free parameters load-bearing?

**Status:** Open for eight parameters. **One has been settled** — not by a sensitivity analysis, but by being superseded.

> **Updated 2026-07-27** on ingest of [[wide-open-spaces-space-creation|Wide Open Spaces]], which adds four. The count has gone four → five → eight across three ingests, and no source has swept any of them.

| Parameter | Framework | Role | Justification given | Status |
|---|---|---|---|---|
| $\gamma = 0.95$/s | [[temporal-discounting\|Shelopugin]] | Credit decay | "Stylistic preference" | **Open** |
| $\epsilon = 15$ s | [[expected-value-possession-framework\|Fernández et al.]] | Hard credit cutoff | Mean possession duration | **Open** |
| $k = 5$ events | [[vdep]] | Lookahead window | Domain intuition | **Open** — inherited by [[gvdep]] |
| $4$ s | [[c-obso]] | Prediction horizon | Accuracy trade-off | **Open** |
| $w = 3$ s | [[space-occupation-gain\|SOG/SGG]] | Gain window | Expert analysts | **Open** |
| $\delta = 5$ m | [[space-occupation-gain\|SGG]] | Closeness threshold | Mean marking distance | **Open** |
| $\alpha = 3$ m | [[space-occupation-gain\|SGG]] | Minimum attraction | Avoids spurious drags | **Open** |
| $\epsilon$ (gain) | [[space-occupation-gain\|SOG/SGG]] | Gain threshold | Excludes ordinary drift | **Open** |
| ~~$C \approx 3.9$~~ | ~~[[vdep]]~~ | ~~Recovery/attack weighting~~ | ~~Event frequency ratio~~ | **Superseded** |

## Four Kinds, Not One List

The count is less informative than the taxonomy. Lumping them together obscures that only some are suspect.

**Horizon parameters** ($\epsilon = 15$s, $k$, 4 s, $w$) bound how far a model looks. Likely **self-limiting**: most credit falls near the event regardless, so extending or shortening the window changes little at the margin.

**Shape parameters** ($\gamma$) govern *how* weight decays rather than *where* it stops. The most suspect: across its author's own proposed range, $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$ — nearly **two orders of magnitude** in the weight on a thirty-second-old action, offered as a stylistic choice for attacking philosophy.

**Geometric thresholds** ($\delta$, $\alpha$) define when a spatial relationship counts. New with this ingest, and different in kind: they do not weight anything, they **gate whether an event is recorded at all**. A 5 m closeness threshold does not scale SGG's magnitude — it decides which draggings exist.

**Detection thresholds** ($\epsilon$ gain) separate signal from drift. Also gating rather than weighting.

**With $C$ gone, the trade-off-weight category is empty.** What remains is one shape parameter, four horizons, and three gates.

## The Gates Are the New Problem

The four SOG/SGG parameters are worse-behaved than the horizons, for a reason worth stating.

A horizon parameter that is slightly wrong produces slightly wrong values. **A gate that is slightly wrong produces the wrong set of events**, and everything downstream is computed on that set. Move $\delta$ from 5 m to 6 m and a different population of dragging incidents enters the metric; the per-player totals are then sums over different things, not differently-weighted sums over the same things.

That makes the gates **less likely to be self-limiting** than the horizons, and it makes rank correlation under a sweep the right test rather than value correlation — see below.

The authors set $\delta$ from *"the minimum distance an opponent is on average to a player in possession"*, which is a principled anchor, and $w$ and $\alpha$ *"alongside expert football analysts"*, which is a defensible source but not a measurement.

## What GVDEP Settled, and How

[[gvdep|GVDEP]] does not sweep $C$. It **removes it**, replacing the frequency-derived constant with weights taken from [[vaep|VAEP]] at the moments ball gains and effective attacks occur — moving both terms from a **frequency scale** to a **score scale**.

The objection this page raised against $C$ — that a frequency ratio encodes how *often* each event happens and says nothing about how much each *matters* — is answered directly. **The vault's only instance of an asserted parameter fixed by principled derivation.**

Two caveats keep it from being clean: the new weights depend on classifiers whose F1 on that data is 0.08–0.15, and GVDEP inherits $k = 5$ untouched.

## Sensitivity Analysis Is Rare, Not Absent

GVDEP sweeps **$n\_nearest$ from 0 to 11**, reporting F1 for all four classifiers at each value — a genuine sensitivity analysis, on *which inputs to include*. [[drso|DRSO]] verifies **five velocity assumptions** against RMSE. [[physics-based-pass-probabilities|Spearman et al.]] fit by MLE grid search with contour intervals.

So the practice exists in this literature and is well understood. **It has simply never been applied to a horizon, shape or gate parameter.** That removes the excuse: groups that sweep one parameter and publish the curve could sweep another.

## The Test

Rank correlation under a parameter sweep. For each, recompute the player or team rankings across a defensible range and report Spearman's $\rho$ between the extremes.

| Result | Reading |
|---|---|
| $\rho > 0.95$ | Not load-bearing. The choice is free and the debate moot |
| $\rho \approx 0.7$–$0.9$ | Rankings shift at the margins — enough to change a shortlist, not a conclusion |
| $\rho < 0.7$ | Every published ranking is one arbitrary choice away from a different answer |

**Rank correlation rather than value correlation**, because the decisions these metrics inform are ordinal: who to sign, who to review, which moment to watch. This matters doubly for the gates, where value correlation would be measuring sums over different event sets.

**For $\gamma$, test the author's own claim.** He asserts 0.9 suits vertical attacking and 0.99 possession play. Do direct-style teams rank higher under 0.9? If so it is a *feature* and should be reported as a family of metrics; if rankings barely move, the stylistic framing is unfounded.

**For $\delta$, check the event count first.** A cheap pre-test: how many dragging incidents does SGG detect at 4, 5 and 6 m? If the count moves sharply, the gate is load-bearing before any ranking is computed.

## Two Routes Nobody Uses

**Fit against a criterion.** Choose the value maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity]] — both already computed for other purposes. The objection, that optimising for reliability might favour a degenerate metric stable because it measures little, is why predictive validity is the better of the two and both should be reported.

**Derive it from an existing model**, as GVDEP does. Available where a parameter expresses a *trade-off between quantities another model already values*. It does not help the horizons or gates: no existing model values "how far back credit should propagate" or "how close counts as marking".

## What Would Change

**Horizons inert, $\gamma$ and the gates not** — the literature's blanket omission is half-excusable, but the parameters that matter have never been examined, and that should be stated on [[pass-carry-reward|PCR]] and [[space-occupation-gain|SOG/SGG]].

**All eight inert** — a useful negative result, and the vault's repeated complaint should be softened.

**Rankings move substantially** — no framework here reports *a* ranking; each reports one point in a family, and cross-framework comparison is confounded by parameter choice on top of everything else.

## Why Nobody Has Done It

A sensitivity analysis can only weaken a paper: it shows results are robust, which reviewers assume anyway, or shows they are not, which invites the question of why *this* value was chosen.

GVDEP is a partial counter-example — it published a sweep and the sweep was **flattering**, showing its method works with fewer inputs than its predecessor assumed. That suggests the asymmetry breaks when the sweep is over something the authors *want* to show is unnecessary, rather than over a value they had to choose.

## See Also

- [[model-selection]] · [[gvdep]] · [[vdep]] · [[temporal-discounting]] · [[space-occupation-gain]]
- [[split-half-reliability]] · [[predictive-validity]] · [[pass-carry-reward]] · [[c-obso]] · [[drso]] · [[obso]]
- [[action-valuation]] · [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Shelopugin]] · [[football-defence-evaluation-vdep|VDEP]] · [[generalized-vdep-euro-location-analysis|GVDEP]] · [[wide-open-spaces-space-creation|Wide Open Spaces]]
