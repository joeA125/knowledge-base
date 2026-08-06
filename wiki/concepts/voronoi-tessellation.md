---
title: "Voronoi Tessellation"
type: concept
tags: [pitch-control, spatiotemporal, sports-analytics, optical-tracking-data, statistics, tactical-analysis, approximation]
sources: [raw/papers/wide_open_spaces_creation_football.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 25%
  generated: 5%
  imported: 15%
  ambiguous: 0%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Voronoi Tessellation

Partitioning a space so each point belongs to whichever seed is nearest.^[imported: the general definition is standard geometry, not from any held source] In sport, the seeds are players and the cells are **dominant regions** — the area each player is closest to.

Introduced to team sports by Taki & Hasegawa (1996) and, until recently, the default model of space ownership. It matters here as **the baseline both of the vault's [[pitch-control]] traditions were built to replace.**

## What It Was Used For

Prior applications, as catalogued by [[wide-open-spaces-space-creation|Fernández & Bornn]]:

- Quantifying attacking and defending dominance in small-sided games (Silva et al., 2014)
- Evaluating space dominance from passing behaviour (Rein et al., 2016)
- Improving pass probability models (Horton et al., 2014)
- Valuing rebound positioning in basketball (Maheswaran et al., 2014)

Extensions added faster computation and motion-weighted variants, but the core stayed the same.

## The Two Objections

**Ownership is discrete when it should be continuous.** A Voronoi cell asserts that one player owns a region outright and the next owns the adjacent one, with a hard boundary between. Real control is graded and uncertain, particularly in the space between two players — precisely where contests are decided.

> *"all the different Voronoi tesselation-based approaches start from the idea of finding regions that are exclusively dominated by a given player. This concept disregards the concept that ownership of space is continuous, not discrete, with uncertainty in who controls areas between players."*

**Distance to the ball is ignored.** Whether a location matters, and how far a player's influence extends over it, depends on where the ball is. A pure proximity partition cannot express that.

## How Each Tradition Replaced It

Both of the vault's pitch-control models are answers to the first objection, reached independently:

| | Mechanism | Continuity comes from |
|---|---|---|
| [[pitch-control\|Fernández & Bornn]] | Gaussian influence, logistic difference | The **density**, and the sigmoid |
| [[pass-probability-model\|Spearman]] | Arrival-time contest, Poisson control | **Temporal uncertainty** $\sigma$ on intercept time |

The routes differ interestingly. Fernández & Bornn make control continuous by making *influence* a smooth function of space. Spearman makes it continuous by making *arrival time* uncertain — the softness enters as a distribution over when a player gets there, not over where he reaches.

Both also address the second objection, and again differently: Fernández & Bornn scale the influence radius by distance to the ball (4 m to 10 m); Spearman models ball flight time explicitly, with drag.

## Why It Is Worth a Page

Not because the vault uses it — nothing here does — but because **the two traditions define themselves against it rather than against each other.**

Neither Spearman nor Fernández & Bornn cites the other. Both cite the Voronoi line. That is a structural explanation for `pitch-control-traditions-uncompared`^[generated: the inference that a shared opponent explains the absence of mutual comparison is drawn here; neither source states it. rests-on: source:fb-voronoi-critique, absence:no-held-source-compares-ppcf-and-gaussian]: the two are **siblings rather than rivals**, each framed as an improvement on a common ancestor, with no reason built into either paper to compare them.

That is worth holding onto when reading the comparison question. Two methods that never engaged are not the same as two methods that engaged and disagreed — the silence is a fact about the field's structure, not about the methods' relative merits.

## See Also

- [[pitch-control]] · [[pass-probability-model]] · [[pitch-control-traditions-compared]] · [[probability-surface]]
- [[space-occupation-gain]] · [[off-ball-value]] · [[optical-tracking-data]] · [[tactical-analysis]]
- [[javier-fernandez]] · [[luke-bornn]] · [[william-spearman]]
- [[wide-open-spaces-space-creation|Source Summary]]
