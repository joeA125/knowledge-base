---
title: "Does tracking error propagate into value estimates?"
type: question
tags: [multi-object-tracking, optical-tracking-data, uncertainty-quantification, pitch-control, probability-surface, evaluation, sports-analytics, needs-review]
sources: [raw/papers/soccernet-game-state-reconstruction.md, raw/papers/beyond_expected_goals.md, raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.7
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Does tracking error propagate into value estimates?

**Status:** Open, and unusually so — the input error characteristics are not merely unmodelled but **unpublished**.

Every tracking-based framework in this vault takes player positions as **given**: [[pitch-control]], [[obso]], [[c-obso]], [[soccermap]], [[martingale-epv]], [[xsot|xOSOT]]. Positions are not given. They are the output of a [[multi-object-tracking|tracker]], with identity switches and localisation error baked in.

**No source propagates tracking uncertainty into downstream value estimates**, and the commercial providers ([[stats-perform]], [[data-stadium]]) report no MOTA, IDF1 or HOTA at all. So the error characteristics of the data underlying every tracking-based finding here are unknown.

## Why This Is Not Just Ordinary Measurement Noise

Three properties make tracking error worse than a generic noise floor.

**Errors are structured, not random.** [[multi-object-tracking|Identity switches]] happen under occlusion, and occlusion happens in crowds. So error concentrates in **exactly the regions where value concentrates** — the penalty area, around the ball, at set pieces. This is also where the two [[pitch-control-traditions-compared|pitch-control traditions]] are predicted to disagree most, which means two independent sources of divergence coincide in the same locations.

**Identity switches corrupt two trajectories at once.** A switch does not lose a player; it swaps two. Any model reading "the nearest defender" or "this attacker's position" gets a coherent but wrong answer, with no signal that anything is amiss.

**Velocity is a derivative and amplifies noise.** [[pitch-control|PPCF]] depends on velocity through the expected intercept time. Differentiating a noisy position series produces a noisier velocity series by construction. The frameworks most dependent on velocity are therefore most exposed — and [[xsot|Yeung & Fujii]] hit the extreme case, where StatsBomb 360 supplies no velocity at all and they set it to zero.

That last case is informative: a framework was published with velocities set to **zero** rather than estimated, and no sensitivity analysis accompanies the choice.

## What Can Be Settled Without Provider Data

The obstacle looks like missing information — nobody publishes tracker error rates. But **a sensitivity curve does not require knowing the true error rate.**

Perturb clean tracking data with synthetic error at a range of magnitudes, recompute the surfaces and metrics, and measure how fast conclusions degrade. That yields a function from error magnitude to conclusion stability. Providers' actual error rates then only need to be located on that curve — and if the curve is flat over any plausible range, the question is closed without them.

## Proposed Test

1. **Positional jitter.** Add Gaussian noise at 0.1 m, 0.25 m, 0.5 m, 1 m to all positions. Recompute [[pitch-control]] and [[obso|OBSO]]. Report surface correlation and, more importantly, **change in player rankings**.
2. **Identity switches.** Swap two same-team players' identities for a window, at realistic rates, concentrated under occlusion. Prediction: far more damaging than jitter of equivalent magnitude, because the error is coherent.
3. **Velocity ablation.** Recompute PPCF with (a) estimated velocities, (b) velocities zeroed as in Yeung & Fujii, (c) velocities with added noise. This directly tests whether their forced simplification is material — a question their own paper raises and does not answer.
4. **Stratify by region.** Prediction: degradation concentrated in crowded areas, so a global average understates the effect on the values that matter. The same stratification logic as [[pitch-control-traditions-compared]].
5. **Cross-provider.** Where the same match exists in two providers' data, compute the metric on both. This measures total end-to-end divergence without needing either provider's error model.

Step 5 is the strongest if the data exists, since it bounds real-world disagreement directly rather than simulating it.

## What Would Change

**If metrics are robust** — a reassuring and publishable negative result, and it would justify the field's current practice of ignoring the issue. Nobody has established this.

**If metrics are sensitive to jitter** — then differences between tracking-based frameworks measured on different providers' data are partly artefacts of data quality, and cross-framework comparison needs provider matching.

**If identity switches dominate** — the relevant provider metric is IDF1 rather than positional accuracy, and the field should be asking for a number providers do not currently publish. That is an actionable procurement point for a club, not just a research finding.

## Why Nobody Has Done It

A division of labour. The [[multi-object-tracking|computer vision]] literature optimises MOTA, IDF1 and HOTA and treats the output as the deliverable. The valuation literature treats positions as the input and does not ask where they came from. The two communities publish in different venues and cite each other only for pipeline context — see [[game-state-reconstruction]], which sits at the join and is the closest thing here to a bridge.

Commercially, the incentive runs the wrong way: a provider gains nothing from publishing its own error rate.

## See Also

- [[multi-object-tracking]] · [[optical-tracking-data]] · [[game-state-reconstruction]] · [[camera-calibration]]
- [[pitch-control]] · [[obso]] · [[c-obso]] · [[probability-surface]] · [[soccermap]] · [[xsot]]
- [[uncertainty-quantification]] · [[stats-perform]] · [[data-stadium]]
- [[pitch-control-traditions-compared]] — errors concentrate where the two traditions also disagree
- [[action-valuation-frameworks-compared]]
