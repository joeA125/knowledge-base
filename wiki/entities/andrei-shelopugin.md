---
title: "Andrei Shelopugin"
type: entity
tags: [person, researcher, independent-researcher, sports-analytics, single-source]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 60%
  inferred: 30%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Andrei Shelopugin

Independent researcher in football analytics, and **sole author** of "Expected Possession Value of Control and Duel Actions for Soccer Player's Skills Estimation" — the [[epv-control-duel-skills-football|EPV control and duel paper]].

Notable in this vault as the only football-analytics author with no institutional or club affiliation. Every other line of work here comes from a university group ([[keisuke-fujii]], [[jesse-davis]], [[universidade-do-porto]]), a research lab, or a club analytics department.

## Line of Work

Three connected papers, the earlier two co-authored with [[alexander-sirotkin]]:

| Year | Work | Contribution |
|---|---|---|
| 2023 | 1v1 abilities via modified Glicko-2 (IEEE PRDC) | The [[duel-skill-rating]] method |
| 2023 | Ratings of European and South American leagues (arXiv) | The [[league-strength-rating]] method |
| — | EPV of control and duel actions | Combines both into a valuation and [[transfer-performance-prediction\|forecasting]] system |

The EPV paper is best read as the capstone: two independently published rating systems are folded in as components, one supplying duel win probabilities as model features, the other supplying club and league strength as forecasting features.

## Characteristic Concerns

Three preoccupations recur and distinguish the work from the rest of the vault's football sources.

**Time as a physical rather than nominal quantity.** [[effective-playing-time|Effective playing time]] and [[temporal-discounting|geometric decay]] both reflect an insistence that elapsed time is the right currency for credit assignment, where other frameworks count actions.

**The events nobody else values.** [[symmetrical-duel-valuation|Symmetrical duels]] are treated as first-class, in a literature that quietly restricts itself to actions with unambiguous possession.

**Selection, stated openly.** The [[positive-unlabeled-learning|presence-only]] discussion is unusually candid — the biases are named, a heuristic patch is offered, and the patch is described as inadequate. This is more forthright than most of the peer-reviewed work in this vault.

The framing throughout is practitioner-oriented: the stated goal is narrowing a scouting pool, not describing a completed season, and the outputs are destination-conditioned shortlists rather than rankings.

## Caveats

The work is a preprint, single-authored and not peer-reviewed. Data current to 1 June 2024; the provider is never named, though the duel taxonomy suggests Wyscout. Acknowledgements thank Nikita Kozodoi, Viktoria Lokteva, Nikita Vasyukhin, Iskander Safiulin, Daniil Babaev, and Alexander Sirotkin for consultation.

Vault knowledge of this person comes from a single paper. Nothing beyond the three cited works is established.

## See Also

- [[epv-control-duel-skills-football|Source Summary]]
- [[pass-carry-reward]] · [[symmetrical-duel-valuation]] · [[duel-skill-rating]]
- [[temporal-discounting]] · [[effective-playing-time]] · [[possession-risk]]
- [[league-strength-rating]] · [[transfer-performance-prediction]]
- [[alexander-sirotkin]]
