---
title: "Dynamic Pressure Lines"
type: concept
tags: [tactical-analysis, clustering, sports-analytics, optical-tracking-data, feature-engineering, spatiotemporal, interpretability]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.75
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Dynamic Pressure Lines

A data-driven representation of team shape: players clustered into the horizontal bands — defenders, midfielders, forwards — that coaches already reason about, recomputed continuously rather than fixed by nominal formation.

## Definition

Given player locations $P = \{p_1, \dots, p_n\}$, the pressure lines are the centroids of a **complete-linkage clustering** of $P$ into $k$ partitions, minimising intra-cluster and maximising inter-cluster distance.

Clustering on the breadth-wise coordinate gives **vertical** pressure lines; on the depth-wise coordinate, **horizontal** ones. The source uses $k = 3$ for both — three vertical lines recovering the conventional defender/midfielder/forward split, and three horizontal ones marking the flanks and the two halves of the central block.

Complete linkage is the right choice here: it produces compact, well-separated clusters, which is what a defensive line *is*. Single linkage would chain players into a band stretching across the pitch.

## Why Not Just Use the Formation

A nominal formation is a pre-match label. What matters tactically is where the lines are **right now** — they compress when a team drops off, stretch when it presses high, and break when a team is pulled out of shape.

The source recomputes them over windows of a few seconds, so the representation tracks the shape a team is actually holding rather than the one it was announced in. The formation is then *derived* from the clustering — counting players per line — rather than assumed.

## What It Buys

Pressure lines are the anchor for a family of features that would otherwise be inexpressible:

- **Relative position.** Which line is the ball behind? A player receiving in front of the first line faces different pressure and turnover risk than one receiving between the second and third.
- **Line-breaking.** Whether a pass crosses a pressure line, and which one. The FC Barcelona analysts who advised the work regarded line-breaking passes as the single biggest driver of goal expectation.
- **Outplayed players.** How many opponents a pass takes out of the game.
- **Inside vs outside the block.** Whether a pass goes into the opponent's shape or around it.

The empirical payoff is one of the more striking results in the source. EPV-added distributions widen monotonically with the line broken: passes breaking the first line cluster near zero, while third-line breaks centre around +0.005 with mass spanning $[-0.025, 0.05]$. **Breaking a deeper line is both more valuable and more dangerous** — a quantification of something coaches assert but could not previously measure.

## Interpretability as a Design Goal

Worth noting what kind of feature this is. Pressure lines are unashamedly **handcrafted**, built in collaboration with professional analysts to encode existing tactical vocabulary.

That sits in tension with a finding elsewhere in the vault: [[understanding-football-possessions-path-signatures|the Sig-Model]] degrades when handcrafted geometry is added, because its representation can recover the geometry itself. See the [[feature-engineering]] discussion.

The tension is real but the goals differ. Sig-Model optimises predictive accuracy, where an engineered feature is at best redundant. This framework optimises for a coach being able to *act* on the output — and a model that reasons in terms of pressure lines can be argued with by someone who thinks in pressure lines. The engineered feature is buying communicability, not accuracy, and the source never claims otherwise.

## Limitations

- **$k = 3$ is imposed.** A team defending in a 5-4-1 and one in a 4-4-2 both get three lines. Selecting $k$ per situation ([[model-selection|by BIC or silhouette]]) is not attempted.
- **One-dimensional clustering.** Vertical and horizontal lines are computed separately, so a diagonal or staggered shape is represented awkwardly.
- **No player identity.** A line is a set of positions; the framework does not know a full-back has stepped into midfield.

## See Also

- [[tactical-analysis]] · [[pitch-control]] · [[off-ball-value]]
- [[clustering]] · [[feature-engineering]] · [[interpretability]]
- [[expected-possession-value]] · [[soccermap]] · [[optical-tracking-data]]
- [[expected-value-possession-framework|Source Summary]]
