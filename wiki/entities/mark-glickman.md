---
title: "Mark Glickman"
type: entity
tags: [person, researcher, university, statistics, bayesian, ranking-system]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/bayesian-true-skill-rating.md]
confidence: 0.7
provenance:
  extracted: 40%
  inferred: 50%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Mark Glickman

Statistician, creator of the [[glicko-rating-system|Glicko]] and Glicko-2 rating systems. Associated with Boston University at the time of the Glicko-2 documentation cited in this vault, and subsequently with Harvard.

## Contribution

Glicko's advance over [[elo-rating-system|Elo]] was to represent a player's skill as a **distribution rather than a point** — a mean $\mu$ paired with a rating deviation $\phi$ quantifying uncertainty. Two consequences follow directly:

- Update size becomes *adaptive*. A result against a well-established opponent moves your rating less than one against an uncertain opponent, and your own uncertain rating moves further than a settled one.
- **Inactivity increases uncertainty.** Rating deviation grows over time without play, so a returning player's rating adjusts quickly rather than being anchored to stale evidence. Elo has no mechanism for this and handles it with ad-hoc "provisional" flags.

**Glicko-2** adds a third parameter, *volatility*, capturing how erratic a competitor's results are — distinguishing a player of stable moderate strength from one of the same mean who swings wildly.

## Influence in This Vault

Glickman's work sits at a junction. [[trueskill]] takes the Gaussian-belief idea and extends it to teams and multiplayer settings via [[factor-graph]] inference. The football line takes it in a different direction entirely, applying Glicko-2 to contests that are not games:

- [[duel-skill-rating]] — individual aerial and ground duels as the contest unit, with a learned advantage term correcting the [[bradley-terry-model|symmetry assumption]].
- [[league-strength-rating]] — clubs as competitors, aggregated to rate whole competitions.

Both modify the update rule rather than adopting it wholesale, which is a reasonable measure of the framework's flexibility.

**Note:** vault knowledge of this person is drawn from citations within papers using his methods, not from primary sources. Biographical detail beyond authorship of Glicko and Glicko-2 is unverified.

## See Also

- [[glicko-rating-system]] · [[elo-rating-system]] · [[trueskill]]
- [[bradley-terry-model]] · [[bayesian-inference]]
- [[duel-skill-rating]] · [[league-strength-rating]]
