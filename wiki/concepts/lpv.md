---
title: "LPV (Location-based Possession Value)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, predictive-validity, event-stream-data, interpretability]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 12%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# LPV (Location-based Possession Value)

LPV ([[understanding-football-possessions-path-signatures|Hirnschall & Bajons, 2025]]) values a football possession by scoring each action with established metrics — [[expected-goals|xG]] and [[expected-threat|xT]] — evaluated at the *predicted* location from the [[sig-model]].

## Definition

Location-based Action Value:

$$\text{LAV} = \widehat{xG} \cdot P(\text{shot}) + \widehat{xT} \cdot P(\text{dribble, pass, cross})$$

$$\text{LPV} = \sum_{i=1}^{N_p} \text{LAV}_i$$

The split is intuitive: if the model thinks a shot is coming, value it by shot quality at that spot; if it thinks the ball will be moved, value it by the threat that spot generates. Both $\widehat{xG}$ and $\widehat{xT}$ are simple models fitted on four European leagues excluding the Premier League, to avoid contaminating the analysis.

## The Critique of HPUS It Answers

LPV is explicitly a response to [[hpus]]. The objection is about **unjustified constants**:

> While HPUS makes use of the location and the elapsed time of the actions, it is not clear how the factors (0, 5, and 10) and the zones were chosen. In fact, Yeung et al. even suggest that these values can be adjusted arbitrarily. However, this makes HPUS difficult to interpret and also questions the reliability of the metric, as different values could potentially lead to different results.

This is a real methodological point: a metric whose scale is arbitrary cannot be compared across studies, and its rankings could shift under a different but equally defensible choice of constants. LPV's values are instead denominated in units practitioners already understand — probability of scoring, and expected threat.

The structural change is also worth noting. HPUS *multiplies* an action score by a location score; LPV makes the value **location-dependent** rather than multiplying by location. So a shot is not "10 discounted by position" but simply the xG at that position.

## What LPV Deliberately Omits

Two HPUS components are dropped, with reasons given:

- **Interevent time.** HPUS divides by elapsed time to reward speed. The authors call the intuition reasonable but note it is asserted without data-driven justification — and [[sig-model]] does not model time anyway.
- **The decay weighting $\phi$.** They accept the principle that later actions matter more, but argue the specific exponential is unjustified, and that weighting matters mainly for long possessions while short ones may weight all actions equally.

Both are left to future work rather than adopted uncritically — a defensible stance, though it means LPV discards information HPUS uses.

## Predicted vs Observed Possession Value

The paper's other methodological contribution. Every metric of this family can be computed two ways:

- **Predicted** — plug in the model's action probabilities.
- **Observed** — plug in 1 for the action that actually occurred, 0 for the rest.

The gap between them measures how effectively a team realises the threat available to it. Predicted value is systematically higher, since it accounts for *all* options at each step while the team took only one — so accumulated suboptimal decisions show up as a larger gap.

Across the 2017/18 Premier League this relative gap correlates $R \approx -0.876$ with final points and $R \approx -0.907$ with season xG: **better teams convert more of their available possession value.** Tottenham stand out as unusually effective by this measure, which the authors offer as partial explanation for their 3rd-place finish.

## Validation

| | poss-util | [[hpus\|HPUS]] | **LPV** | xG | goals |
|---|---|---|---|---|---|
| vs same-match xG | 0.43 | 0.41 | **0.51** | — | 0.68 |
| vs same-match goals | 0.16 | 0.19 | **0.33** | 0.68 | — |
| vs **next-match** xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs **next-match** goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

LPV leads on every measure. The bottom two rows are the interesting ones — see [[predictive-validity]]. Both HPUS and LPV forecast next-match performance **better than xG or goals do**, meaning possession-value metrics capture something about underlying team quality that scoreline outcomes miss.

Aggregated over the season, LPV correlates $R \approx 0.896$ with final points and $R \approx 0.934$ with total xG.

## A Caveat the Authors Raise Themselves

Comparing LPV against xG may look unfair, since LPV *contains* an xG term. They defend it on the grounds that LPV's xG is a simple logistic regression on distance and goal angle (following Pollard & Reep, 1997), while the comparison xG comes from understat's neural network trained on a far larger dataset with many more features.

## See Also

- [[hpus]]
- [[sig-model]]
- [[expected-goals]]
- [[expected-threat]]
- [[action-valuation]]
- [[predictive-validity]]
- [[action-valuation-frameworks-compared]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
