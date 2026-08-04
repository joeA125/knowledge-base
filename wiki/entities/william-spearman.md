---
title: "William Spearman"
type: entity
tags: [person, researcher, practitioner, sports-analytics, off-ball, pitch-control, optical-tracking-data, theory-based-modelling]
sources: [raw/papers/physics_based_pass_probabilities.md, raw/papers/beyond_expected_goals.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 17%
  generated: 2%
  imported: 1%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# William Spearman

Creator of the **physics-based pass probability model**, the **potential pitch control field**, and [[obso|OBSO]] — between them the substrate for more of this vault's football coverage than any other single line of work.

**Affiliation: Hudl**, stated on both held papers (2017 and 2018).

> **Correction, resolved.** An early revision of this page claimed a Liverpool FC research affiliation, imported from background knowledge rather than any source. Now doubly refuted by two primary sources. Nothing in `raw/` supports it and it is not asserted here.

## Two Held Sources

| Year | Work | Contribution |
|---|---|---|
| 2017 | [[physics-based-pass-probabilities\|Physics-Based Modeling of Pass Probabilities]] (with Basye, Dick, Hotovy, Pop) | The intercept/control model; **PPCF**; receiving efficiency; pass value; hypothetical passing |
| 2018 | [[beyond-expected-goals\|Beyond Expected Goals]] | [[obso\|OBSO]] — transition × control × score |

The order matters and the vault had it backwards. **Pitch control is not OBSO's control term; OBSO is an application of a pass-reception model built a year earlier.** See [[pitch-control]].

## The Methodological Signature

**Physical rather than statistical.** Arrival times come from acceleration limits, ball flight from aerodynamic drag, control from a Poisson process. Parameters carry units — seconds, hertz — so priors can be set from measurement rather than taste.

Two consequences the vault has repeatedly found significant:

**Parameters are fitted with stated errors.** The 2017 paper reports $\sigma = 0.45 \pm 0.01\,\text{(stat)} \pm 0.04\,\text{(syst)}$ and $\lambda = 4.30 \pm 0.28 \pm 1.1$ — **separate statistical and systematic uncertainties, which almost nothing else in this vault reports.** The 2018 paper then refits by MAP using those values as priors. This is the basis of `physical-units-admit-priors`^[generated: declared on [[model-selection]]. rests-on: claim:physical-units-admit-priors] — and the 2017 paper strengthens it, since the priors are demonstrably inherited from a prior measurement.

**Reproducibility as a design constraint.** OBSO needs ~1,000 frames per match and no ball tracking. Compare [[martingale-epv|Cervone et al.'s]] 461 processors for the same category of question.

## Validation, Which Is Unusual Here

The 2017 model is checked against **who actually received 5,471 held-out passes** — 81% accuracy on the receiving team, 68% on the specific player. A directly observable ground truth, which most of this literature lacks.

The 2018 paper adds the vault's strongest [[predictive-validity]] result: **OBSO predicts a player's next-match goals (0.26) better than his shots (0.17) or goals (0.12) do.**

Note the pair covers both validation modes — a component checked against observable outcomes, and a derived metric checked against future ones. Few authors here do either; nobody else does both.

## Why He Is Central

Three independent lines depend on this work:

- The **[[keisuke-fujii|Fujii group]]** builds [[c-obso]] and [[drso]] on OBSO, and PPCF underlies [[xsot|xOSOT]].
- **[[expected-value-possession-framework|Fernández, Bornn & Cervone]]** cite Spearman as the closest prior work on off-ball valuation; their [[pitch-control|Gaussian influence]] model is an alternative to his.
- The vault's [[prescription|prescriptive task]] traces to his **hypothetical passing** analysis (2017), which predates [[xsot|Yeung & Fujii]] by seven years.

## A Citation Note

[[c-obso|Teranishi et al.]] and [[drso|Umemoto & Fujii]] both use $\sigma = 0.45$, $\lambda = 4.30$ while citing the **2018** paper, which fits 0.54 and 3.99. The values are the **2017** fits — legitimate, misattributed. See [[obso]].

## Two Pitch-Control Traditions

| | Spearman | [[pitch-control\|Fernández & Bornn]] |
|---|---|---|
| Origin | A pass-reception model | A spatial-dominance model |
| Mechanism | Arrival-time contest, Poisson control | Gaussian influence density |
| Grounding | **Physical** | Statistical |
| Parameters | **Fitted, with stated errors** | Set to 1 |
| Directly validated | **Yes — 81%/68% on held-out passes** | No |

**No source compares them**, and both feed value models whose outputs are compared. See [[pitch-control-traditions-compared]].

## See Also

- [[obso]] · [[pitch-control]] · [[off-ball-value]] · [[c-obso]] · [[drso]] · [[space-creation]]
- [[expected-goals]] · [[probability-surface]] · [[predictive-validity]] · [[theory-based-modelling]] · [[model-selection]]
- [[keisuke-fujii]] · [[javier-fernandez]] · [[luke-bornn]] · [[rikuhei-umemoto]]
- [[physics-based-pass-probabilities|2017 Summary]] · [[beyond-expected-goals|2018 Summary]]
