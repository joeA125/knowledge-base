---
title: "Alexander Sirotkin"
type: entity
tags: [person, researcher, sports-analytics, ranking-system, single-source, stub]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.55
provenance:
  extracted: 40%
  inferred: 40%
  ambiguous: 20%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Alexander Sirotkin

Co-author with [[andrei-shelopugin]] of the two rating-system papers that the [[epv-control-duel-skills-football|EPV control and duel paper]] builds on:

- **"Evaluating of football player 1v1 abilities based on the Glicko-2 with modifications"** (IEEE PRDC, 2023) — the basis of [[duel-skill-rating]].
- **"Ratings of European and South American football leagues based on Glicko-2 with modifications"** (arXiv, 2023) — the basis of [[league-strength-rating]].

Both apply the same underlying idea — [[glicko-rating-system|Glicko-2]] with structural modifications — at two different scales: individual players contesting duels, and clubs contesting matches. That the pair pursued the same methodological line at both levels is what makes the EPV paper's architecture possible, since player-level and competition-level ratings end up on compatible footings.

Sirotkin is not an author of the EPV paper itself but is thanked in its acknowledgements for consultation on machine learning and soccer analytics.

**Note:** vault knowledge of this entity comes only from citations and acknowledgements within a single paper. Affiliation, and any work outside these two papers, is unknown.

## See Also

- [[andrei-shelopugin]]
- [[duel-skill-rating]] · [[league-strength-rating]]
- [[glicko-rating-system]]
- [[epv-control-duel-skills-football|Source Summary]]
