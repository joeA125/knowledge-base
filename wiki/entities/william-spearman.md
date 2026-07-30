---
title: "William Spearman"
type: entity
tags: [person, researcher, practitioner, sports-analytics, off-ball, pitch-control, optical-tracking-data]
sources: [raw/papers/beyond_expected_goals.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# William Spearman

Creator of [[obso|OBSO]] (off-ball scoring opportunity) and the physics-based **potential pitch control field** underlying it.

**Affiliation: Hudl**, stated on [[beyond-expected-goals|Beyond Expected Goals]] (MIT Sloan, 2018), whose data comes from Hudl's own tracking and event collection.

> **Correction, 2026-07-27.** An earlier revision of this page stated he was "associated with Liverpool FC's research department." That was inferred from the wider literature rather than from any held source, and is not supported by the primary source now held. He is widely reported to have moved to Liverpool subsequently, but nothing in `raw/` establishes that, and it should not be asserted here.

## Two Contributions

**Physics-based pass and control modelling** (Spearman, Pop, Basye, Hotovy & Dick, 2017). Treats a player's ability to control the ball as a **Poisson point process**, with control probability accumulating the longer a player is near the ball uncontested and shared competitively across all players. This is the PPCF term inside OBSO. Not held in `raw/`; it supplies the priors for two of the 2018 parameters.

**[[obso|OBSO]]** (2018). Factorises the value of an off-ball position into transition × control × score, each a separately modelled surface, combined by the chain rule.

## The Methodological Signature

What distinguishes his work from the rest of the vault's off-ball literature is that **the models are physical rather than statistical.** Arrival times come from acceleration limits; ball flight comes from aerodynamic drag; control is a Poisson process. Parameters have units — seconds, hertz, metres — which means priors can be set from measurement rather than from taste, and MAP estimation on five training matches is enough.

Two consequences worth noting:

- **Cheapness.** OBSO needs ~1,000 frames per match rather than 25 Hz throughout, and no ball tracking at all. Compare [[martingale-epv|Cervone et al.'s]] 461 processors for the same category of question.
- **Reproducibility as a design constraint.** Data requirements are deliberately minimised so the analysis transfers across providers — an explicit goal, rarely stated elsewhere in this literature.

## Why He Is Central Here

Two independent lines depend on his work:

- The **[[keisuke-fujii|Fujii group]]** builds both its counterfactual methods on OBSO — [[c-obso]] for attacking space creation, and Umemoto & Fujii (2023) for defensive positioning.
- **[[expected-value-possession-framework|Fernández, Bornn & Cervone]]** cite this paper as the closest prior work on off-ball valuation, and their [[pitch-control]] construction is an alternative to his.

## Two Pitch-Control Traditions

The vault holds both, and they differ in kind:

| | Spearman | [[pitch-control\|Fernández & Bornn]] |
|---|---|---|
| Mechanism | Arrival-time contest, Poisson control | Gaussian influence density |
| Grounding | **Physical** — time to reach, time to control | Statistical |
| Competition | Explicit, via shared probability mass | Difference of summed influences |
| Asymmetry | **$\kappa = 1.72$ defensive advantage** | None |
| Parameters | Fitted with measured priors | Set to 1, unfitted |

Spearman's is more principled about *why* control occurs, and the shared-mass term makes control correctly zero-sum where summed Gaussian influence over-counts overlapping coverage. **No source in this vault compares them**, and both feed value models whose outputs are compared.

## See Also

- [[obso]] · [[pitch-control]] · [[off-ball-value]] · [[c-obso]] · [[space-creation]]
- [[expected-goals]] · [[probability-surface]] · [[predictive-validity]] · [[structured-model-decomposition]]
- [[keisuke-fujii]] · [[javier-fernandez]] · [[luke-bornn]]
- [[beyond-expected-goals|Source Summary]]
