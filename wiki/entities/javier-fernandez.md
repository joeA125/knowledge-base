---
title: "Javier Fernández"
type: entity
tags: [person, researcher, sports-club, sports-analytics, pitch-control, off-ball, space-creation, probability-surface, deep-learning]
sources: [raw/papers/wide_open_spaces_creation_football.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 72%
  inferred: 24%
  generated: 2%
  imported: 2%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# Javier Fernández

Analyst at [[fc-barcelona]] and lead author of the vault's Gaussian [[pitch-control]] tradition and the decomposed soccer EPV framework.

**Two held sources.**

| Year | Work | With | Contribution |
|---|---|---|---|
| 2018 | [[wide-open-spaces-space-creation\|Wide Open Spaces]] | [[luke-bornn\|Bornn]] | [[pitch-control]] (Gaussian), [[pitch-value-model]], [[space-occupation-gain\|SOG/SGG]] |
| 2020/21 | [[expected-value-possession-framework\|Soccer EPV framework]] | Bornn, Cervone | Decomposed EPV, [[soccermap]], [[dynamic-pressure-lines]] |

## The Progression

The 2018 paper builds the spatial primitives; the 2020 framework builds a value model on top of them. [[pitch-control]] appears in both — as the substrate for space metrics in the first, and as an input feature plus the definition of *pressure* (control below 0.4) in the second.

What changes between them is the **source of value**. In 2018, value comes from [[pitch-value-model|where defenders stand]] — a revealed-preference target needing no outcome labels. In 2020, it comes from [[structured-model-decomposition|nine components fitted to observed outcomes]]. The earlier approach is cheaper and assumption-heavy; the later one is expensive and validated per component by [[probability-calibration|calibration]].

## Club Analysts as Co-Designers

Unusual and worth noting. FC Barcelona analysts are not merely acknowledged in either paper — they **shape the models**:

- The influence-radius range (4–10 m) is set *"based on the opinion of expert soccer analysts"*.
- The three-second window and the 5 m closeness threshold are chosen *"alongside expert football analysts from F.C. Barcelona"*.
- Validation of SOG and SGG is **expert video review by two analysts**, since no ground truth exists for space quantification.
- The 2020 framework's contextual features are similarly club-informed.

That places this line at one end of a spectrum the vault holds: **domain judgement encoded as parameters** at one end, [[physics-based-pass-probabilities|physical measurement fitted by MLE]] at the other. Expert-set parameters cannot inherit priors from prior measurement, but they are inspectable and disputable in a way fitted ones are not. See [[model-selection]].

## The Interpretability Argument, Made Twice

Both papers argue that a model's components should each answer a recognisable football question — 2018 by composing control × value, 2020 by decomposing EPV into nine parts along pass/drive/shoot and success/failure axes.

[[obso|Spearman]] makes the same argument independently in 2018, and far more cheaply. Three papers, two research lines, one design principle: **components should be individually meaningful, not merely jointly predictive.** See [[structured-model-decomposition]] and [[interpretability]].

## What the 2018 Work Left Unfinished

[[space-occupation-gain|SOG and SGG]] have not been extended by anyone, including the authors. The pitch-control model became infrastructure for the 2020 framework and for others; the space metrics did not.

Two plausible reasons, neither established: the metrics rest on **four asserted parameters** and a one-match analysis, and SGG attributes credit by **spatial co-occurrence** rather than by any causal test — a weakness [[c-obso]] addresses four years later from an unrelated group. See [[space-creation]].

## See Also

- [[pitch-control]] · [[pitch-value-model]] · [[space-occupation-gain]] · [[space-creation]] · [[soccermap]]
- [[expected-possession-value]] · [[probability-surface]] · [[structured-model-decomposition]] · [[dynamic-pressure-lines]] · [[off-ball-value]]
- [[single-pixel-supervision]] · [[probability-calibration]] · [[interpretability]] · [[model-selection]] · [[voronoi-tessellation]]
- [[luke-bornn]] · [[daniel-cervone]] · [[william-spearman]] · [[fc-barcelona]]
- [[wide-open-spaces-space-creation|Wide Open Spaces]] · [[expected-value-possession-framework|Soccer EPV Framework]]
