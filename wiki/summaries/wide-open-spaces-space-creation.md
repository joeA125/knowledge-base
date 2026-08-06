---
title: "Wide Open Spaces: A Statistical Technique for Measuring Space Creation in Professional Soccer (Fernández & Bornn, 2018)"
type: source_summary
tags: [summary, sports-analytics, off-ball, space-creation, pitch-control, probability-surface, optical-tracking-data, player-evaluation, tactical-analysis, deep-learning]
sources: [raw/papers/wide_open_spaces_creation_football.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 12%
  generated: 2%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Wide Open Spaces: A Statistical Technique for Measuring Space Creation in Professional Soccer

**Authors:** [[javier-fernandez]] ([[fc-barcelona]]) · [[luke-bornn]] (Simon Fraser University, Sacramento Kings)
**Venue:** MIT Sloan Sports Analytics Conference, 2018
**Data:** Metrica Sports optical tracking. 20 Spanish first and second-B division matches (2.4M examples) for the value model; one Barcelona–Villarreal match (Jan 2017, 1–1) for the analysis.

**The origin of the vault's second [[pitch-control]] tradition**, and the last unheld paper in a chain the vault has depended on for months. Long carried as acquisition priority 1.

## Three Models, Composed

**Player influence.** A bivariate normal whose shape adapts to position, velocity and distance to the ball, normalised against its own peak: $I_i(p,t) = f_i(p,t) / f_i(p_i(t), t)$.

The covariance is built by rotation and scaling: rotate to the velocity angle, stretch along it by a speed ratio $s^2/13^2$ (13 m/s assumed maximum), and set the base radius from distance to the ball — **4 m when on the ball, rising to 10 m at 20 m away**. The mean is translated forward by half the velocity vector.

The distance-to-ball rule inverts the naive intuition and the paper argues it carefully: a player *far* from the ball has a *wider* influence, because if the ball travels toward him he has more time to reach a larger area.

**Team pitch control.** The logistic of the difference of summed influences:

$$PC(p, t) = \sigma\Big(\sum_i I_i(p,t) - \sum_j I_j(p,t)\Big)$$

**Ball-relative pitch value.** A feed-forward network learning what each location is worth given ball position — see [[pitch-value-model]]. The training signal is the *defending team's* summed influence, on the hypothesis that defenders position themselves over valuable space. Output is then normalised by distance to goal.

**Quality of owned space** is the product: $Q_i(t) = PC_i(t)\,V(t)$.

## SOG and SGG

The metrics the vault has referenced for months without holding. See [[space-occupation-gain]].

**Space Occupation Gain** — mean change in $Q$ over a three-second window, thresholded at $\epsilon$ so ordinary drift is not counted. Split by speed into **active** (>1.5 m/s) and **passive** occupation.

**Space Generation Gain** — a logical condition on dragging: an opponent starts near teammate $i'$, ends near generator $i$, and leaves $i'$, with a minimum attraction distance. Credit for the freed space goes to the generator.

## Findings

**Messi's passive occupation exceeds his active occupation**, uniquely among the outfield players (66.7% passive against a squad norm nearer 40%). The paper reads the walking directly: *"that walking behaviour is not a detachment from the match but a conscious action to move through empty spaces of value and claim the control of valuable space."* 71% of his gain occurs in front of the ball.

**Occupation and generation are different skills.** Iniesta, Busquets and Messi lead occupation (41% of team SOG between them); Neymar and Suárez lead generation while sitting mid-table on occupation. Suárez generates most inside the box.

**Full-backs generate almost nothing** — moving toward touchlines drags nobody.

**The generator–receiver matrix shows structure**, including a Suárez↔Messi reciprocal pair and Busquets receiving from nearly everyone, which the authors read as third-man-pass behaviour.

## What This Settles for the Vault

### The saturation analysis was right — and it is the authors' own point

The vault generated a claim that the Gaussian model saturates through the sigmoid on a *difference*, so a lone player does not control his own location outright. The paper states it directly:

> *"a single player without any influence of any other player at its current location only controls $\text{logistic}(1) = 0.73$ of the space. This provides the need of higher density of players near a given area to provide higher level of control in that area."*

**That is design intent, not an artefact.** The reading was correct and should be reclassified from generated to extracted on [[pitch-control]].

### "Parameters set to 1, unfitted" was too crude

The vault recorded the Gaussian model as having parameters simply set to 1. More precisely:

- $\gamma$ is an acknowledged simplification — *"we can include a constant within $\sigma$ to add more flexibility, if desired"*, and Equation 2 is described as *"a simplified version"*.
- The influence radius function (4→10 m) **is** parameterised, *"based on the opinion of expert soccer analysts"*.
- Maximum speed 13 m/s, jogging threshold 1.5 m/s.

So the parameters are **expert-set rather than fitted** — a third position between Spearman's MLE fits and "unfitted". That distinction matters for [[model-selection]]: expert-set parameters cannot inherit priors from measurement, but they are not arbitrary either.

### The validation claim needs softening

The vault recorded the Gaussian tradition as "validated against nothing directly." Too strong. The authors state plainly that **no ground truth exists** for space quantification, and validate through *"extensive validation of the developed concepts through video and studying individual situations within games, with the help of two expert soccer video analysts from F.C. Barcelona."*

So: **qualitative expert validation, not none.** The asymmetry with [[pass-probability-model|Spearman's]] 81%/68% against actual pass receivers survives and is better stated — one tradition validates against an *observable outcome*, the other against *expert judgement*, because its target quantity has no ground truth.

### The two traditions never engaged

This paper does not cite Spearman. It positions against **Voronoi tessellation** — see [[voronoi-tessellation]] — as do all its cited predecessors. Spearman likewise does not cite this line.

Both origins are now held, and **neither cites the other.** `pitch-control-traditions-uncompared` is therefore not merely unresolved but structurally explained: the traditions share an opponent (discrete dominance regions) and have no contact with each other.

### Four more asserted free parameters

$w = 3$ s, $\delta = 5$ m, $\alpha = 3$ m, and the $\epsilon$ gain threshold — none swept. The vault's count of asserted parameters without sensitivity analysis rises from four to eight. See [[free-parameters-load-bearing]].

### A shared motive, independently arrived at

The authors state a design goal of *"a model that could be applied in a given data frame, without requiring significant data for learning its parameters"*, citing Spanish-league clubs without tracking access and *"easier reproducibility."*

That is [[physics-based-pass-probabilities|Spearman's]] motive almost exactly, reached independently. **Both pitch-control traditions were built to be cheap and reproducible**, which is a stronger claim about the field's needs than either paper alone supports.

## Assessment

**Strengths.** Three models composing cleanly into one quantity. The pitch-value inversion — learn what is valuable from where defenders stand — is genuinely novel and needs no outcome labels. Expert-analyst involvement is substantive rather than decorative, shaping thresholds and validating situations. And the Messi finding is the rare case of a metric contradicting a common criticism with a mechanism.

**Weaknesses.**

- **One match analysed.** All player findings rest on Barcelona–Villarreal, 845 situations.
- **No ground truth, acknowledged.** Validation is expert video review.
- **Four asserted parameters**, unswept.
- **Space generation is a logical predicate**, not a model — a defender who leaves for unrelated reasons still credits the "generator".
- **Unoccupied generated space is explicitly excluded**, so the metric captures only dragging that benefits a specific teammate.
- **No [[split-half-reliability|reliability]] figure**, and one match cannot supply one.
- The pitch-value model needed **post-hoc normalisation by distance to goal** because it did not learn that value rises up the field — an admitted gap papered over rather than fixed.

## See Also

- [[space-occupation-gain]] · [[pitch-value-model]] · [[voronoi-tessellation]] · [[pitch-control]] · [[space-creation]]
- [[off-ball-value]] · [[c-obso]] · [[obso]] · [[probability-surface]] · [[expected-value-possession-framework]]
- [[javier-fernandez]] · [[luke-bornn]] · [[fc-barcelona]] · [[william-spearman]]
- [[pitch-control-traditions-compared]] · [[free-parameters-load-bearing]] · [[action-valuation-frameworks-compared]]
