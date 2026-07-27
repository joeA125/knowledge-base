---
title: "Garry Gelade"
type: entity
tags: [person, practitioner, sports-analytics, paired-comparison, single-source, stub]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.55
provenance:
  extracted: 35%
  inferred: 45%
  ambiguous: 20%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Garry Gelade

Football analytics practitioner, author of a one-versus-one ability metric published through Stats Perform rather than an academic venue.

## The Contribution

Gelade applied the [[bradley-terry-model]] to duel outcomes, inferring a latent 1v1 strength for each player from who they beat and who beat them. The [[epv-control-duel-skills-football|EPV paper]] credits this as a genuine advance over the win-percentage metrics it replaced, on two grounds:

1. It accounts for **opponent strength**, so a high win rate against weak opposition is not mistaken for ability.
2. The resulting ratings are **transferable across leagues** — a shared latent scale lets players who never meet be compared through chains of common opponents.

[[andrei-shelopugin|Shelopugin's]] [[duel-skill-rating]] is a direct successor, retaining the paired-comparison framing while replacing Bradley-Terry with a modified [[glicko-rating-system|Glicko-2]] to add uncertainty tracking and, more importantly, a contextual advantage term that Bradley-Terry's symmetry assumption cannot express.

## A Recurring Pattern

Gelade is the second practitioner in this vault whose non-academic work became a research baseline — [[karun-singh]], creator of [[expected-threat|xT]], is the other. Both published outside peer review and were subsequently treated as the reference point that academic work positions itself against.

Worth noting when assessing the field: a meaningful share of football analytics' foundational ideas entered through blogs and vendor publications rather than journals, and the citation graph accordingly runs through sources with no formal review.

**Note:** vault knowledge of this person comes only from a citation within a single paper. Nothing beyond the 1v1 metric is established.

## See Also

- [[bradley-terry-model]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[karun-singh]] · [[expected-threat]]
- [[epv-control-duel-skills-football|Source Summary]]
