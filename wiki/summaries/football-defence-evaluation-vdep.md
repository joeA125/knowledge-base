---
title: "Evaluation of Soccer Team Defense Based on Prediction Models of Ball Recovery and Being Attacked (VDEP)"
type: source_summary
tags: [summary, sports-analytics, defensive-valuation, action-valuation, off-ball, optical-tracking-data, event-stream-data, proxy-target, class-imbalance, gradient-boosting, feature-attribution, evaluation, predictive-validity]
sources: [raw/papers/football_defence_evaluation.md]
confidence: 0.85
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Evaluation of Soccer Team Defense Based on Prediction Models of Ball Recovery and Being Attacked

**Authors:** [[kosuke-toda]] · [[masakiyo-teranishi]] · [[keisuke-kushiro]] · [[keisuke-fujii]] (senior)
**Published:** PLOS ONE 17(1): e0263051, 27 January 2022. Open access. Explicitly framed as a **pilot study**.

The first source in this vault to attack **defensive valuation** directly — the gap every other framework here lists among its shared limitations.

## The Argument

Defence is hard to evaluate for a reason that is structural rather than technical: the thing defenders are trying to prevent almost never happens. In 45 matches there were **106 goals** against **9,408 ball recoveries** and **3,701 effective attacks**.

A model trained to predict conceding is therefore learning from an extremely rare positive class, and — the paper's central empirical claim — it does not learn anything usable. The proposal is to give up on the rare event and predict **frequent proxies** instead. See [[rare-event-proxy-targets]].

## VDEP

*Valuing Defense by Estimating Probabilities* — deliberately parallel to [[vaep]], and built by modifying its published code.

Two classifiers predict, over the next $k = 5$ events:

- $P_{recoveries}(S_i)$ — will the defending team win the ball back?
- $P_{attacked}(S_i)$ — will the defending team concede an *effective attack*?

Combined into a single defensive value:

$$V_{vdep}(S_i) = P_{recoveries}(S_i) - C \cdot P_{attacked}(S_i)$$

with $C \approx 3.9$, set from the frequency ratio $9{,}408 : 3{,}701$. Team value is the mean over that team's events in a match.

**Definitions.** An *effective attack* is an event chain ending in a shot **or** penetrating the penalty area — chosen over "shot" deliberately, since a pass instead of a shot in the box is still a defensive failure. A *ball recovery* is a change of attacking team not caused by an effective attack.

## The Off-Ball Feature Set

The other half of the contribution, and the part that matters most to this vault. Where [[vaep]] uses only on-ball action features, VDEP's state $s_i = [a_i, o_i]$ adds an explicit **off-ball** component:

- All 22 players' $x$ and $y$ coordinates (44 dims)
- Each player's distance from the ball (22 dims), **sorted by proximity to the ball**
- Opponent's season goals scored (1 dim), as a crude attacking-strength control

Total 139 dimensions across the current and previous event. The sorting is the quiet design choice — it makes the representation permutation-invariant over players, so "the nearest defender" is always the same feature slot regardless of who it is.

## Results

### The classifiers

| | AUC | Brier | **F1** |
|---|---|---|---|
| $P_{recoveries}$ (VDEP) | 0.770 | 0.184 | **0.522** |
| $P_{attacked}$ (VDEP) | 0.862 | 0.079 | **0.484** |
| $P_{scores}$ (VAEP) | 0.698 | 0.007 | 0.201 |
| $P_{concedes}$ (VAEP) | 0.701 | 0.003 | **0.000** |

**VAEP's conceding classifier has an F1 score of exactly zero.** On this dataset it identifies no true positives at all — it has learned to predict "no goal" always, which is right 99.2% of the time and useless. The difference from VDEP is statistically significant ($F = 144.40$, $p < 10^{-6}$).

Note the inversion between metrics: VAEP wins on **Brier**, VDEP wins on **AUC** and overwhelmingly on **F1**. The paper's explanation is that Brier and accuracy are inflated by huge true-negative counts, so a model predicting the rarer event scores better simply for being right about nothing happening. This is a clean worked example of why metric choice under extreme imbalance is not a detail — see [[class-imbalance-evaluation]].

### Correlation with outcomes

| | Match winning points | Match goals scored | Season winning points | Season goals conceded |
|---|---|---|---|---|
| $R_{vdep}$ | 0.464 | 0.392 | **0.397** | −0.291 |
| $S_{vaep}$ | **0.830** | **0.953** | 0.177 | −0.098 |

Two findings, and the second is more interesting than the first.

**VAEP predicts the match it is measuring, and little beyond it.** Its 0.953 correlation with goals scored in the same match is near-tautological — it is built from a scoring model. Its correlation with *season* points drops to 0.177.

**VAEP has essentially no relationship with goals conceded** ($r = -0.098$ season, $-0.040$ match), despite being explicitly constructed from a conceding classifier. Given that classifier's F1 of 0.000, this is coherent rather than surprising: the conceding half of VAEP is not contributing signal.

**VDEP's match-level and season-level correlations are similar** (0.464 and 0.397), where VAEP's diverge sharply. The authors read this as stability — VDEP measures something persistent about a team rather than reproducing the scoreline.

### Interpretability

[[shap]] attribution identifies the top contributors:

- For $P_{recoveries}$ — the **distance from the ball of the nearest defender**, then whether possession changed on the previous event.
- For $P_{attacked}$ — the **$x$-coordinate of the nearest attacker** (i.e. how advanced), and that attacker's displacement over the event.

Both are off-ball quantities, which is direct evidence the extra feature set is doing work rather than padding.

## Worked Examples

**A conceded goal (Yokohama F. Marinos vs FC Tokyo).** VDEP stays positive across the whole sequence, indicating the defence was not broadly poor. The drops localise to two moments — a forward pass through the line, and a ball carrier left unpressured. The claim is diagnostic granularity: *where* in a sequence the defence failed, not merely that it did.

**A match whose result misleads.** Yokohama won 3–0, yet Tokyo's $R_{attacked}$ (0.116 vs 0.159) and $R_{vdep}$ (0.049 vs −0.040) were both better. Shot counts were equal. The reading is that Tokyo defended well and lost to finishing quality — exactly the case where scoreline and performance diverge.

**Season profiles.** Plotting $R_{recoveries}$ against $R_{attacked}$ separates defensive *styles*: Yokohama recovers often but concedes attacks often (high-risk pressing), while Hiroshima — fewest goals conceded in the league — is strong on both axes.

## Assessment

**Strengths.** The rare-event critique is well-evidenced and generalises well beyond football. The F1-versus-Brier inversion is a genuinely useful methodological demonstration. The off-ball feature construction is simple, permutation-invariant and shown by SHAP to matter. And the honesty is notable — it is labelled a pilot study, the arbitrary constant is flagged by the authors themselves, and unfavourable metrics are reported rather than omitted.

**Weaknesses.**

- **Team-level only.** VDEP cannot evaluate individual defenders, which the authors state plainly as the main limitation. So the vault's individual-defender gap is *not* closed by this.
- **$C \approx 3.9$ is arbitrary.** Set from event frequency, which encodes no view about whether a recovery is genuinely worth 3.9 effective attacks prevented. The authors call this controversial and defer it.
- **45 matches, five weeks, one league.** Very small, and the cross-validation folds are five one-week blocks assumed independent.
- **No ground truth**, acknowledged — the correlational validation is all there is.
- **$k = 5$ chosen by domain knowledge**, with the trade-off described but no sensitivity analysis. The same unjustified-horizon-parameter criticism this vault makes of [[temporal-discounting|Shelopugin's $\gamma$]] and [[expected-value-possession-framework|Fernández et al.'s $\epsilon$]].
- **The VAEP comparison is not quite like-for-like.** VDEP uses $k=5$, VAEP $k=10$, and the two predict different events on a dataset far smaller than VAEP's original. The authors note this. It weakens "VDEP beats VAEP" but not the underlying point, which is that goal-prediction classifiers fail on small data.

## Why It Matters Here

This is the vault's first framework where **preventing** value is the target rather than creating it. Every other approach values defence only implicitly, as the negative half of an attacking model — and this paper shows that half empirically contributing nothing.

It also supplies a general methodological move worth carrying beyond sport: when the outcome you care about is too rare to learn, find a frequent event on its causal path and predict that instead. [[hpus]] arrives at the same place from a different direction, using no goal data at all and still recovering most of xG's signal.

## See Also

- [[vdep]] · [[defensive-valuation]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[vaep]] · [[action-valuation]] · [[off-ball-value]] · [[expected-goals]]
- [[shap]] · [[interpretability]] · [[gradient-boosting]]
- [[keisuke-fujii]] · [[kosuke-toda]] · [[masakiyo-teranishi]] · [[keisuke-kushiro]]
- [[nagoya-university]] · [[kyoto-university]] · [[data-stadium]]
- [[action-valuation-frameworks-compared]]
