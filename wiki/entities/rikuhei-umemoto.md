---
title: "Rikuhei Umemoto"
type: entity
tags: [person, researcher, university, sports-analytics, defensive-valuation, off-ball, counterfactual]
sources: [raw/papers/defensive_player_location_analysis.md, raw/papers/team_defense_positioning_statsbomb.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  generated: 3%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Rikuhei Umemoto

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Lead author of **both** papers in the [[keisuke-fujii|Fujii group's]] defensive-positioning line.

| Year | Work | Contribution |
|---|---|---|
| 2022 | [[generalized-vdep-euro-location-analysis\|GVDEP]] (with [[kazushi-tsutsui]] and Fujii) | [[gvdep]] — score-scaled weights; partial-observation analysis |
| 2023 | [[team-defense-positioning-counterfactuals\|DRSO]] (with Fujii) | [[drso]] — per-defender counterfactual positioning |

## The Two Papers Are Different Approaches, Not Iterations

Worth stating plainly, because the vault conflated them for several entries.

**[[gvdep|GVDEP]]** stays inside [[vdep|VDEP's]] paradigm: predict frequent defensive events with classifiers, then combine the probabilities. Its contributions are a **principled weight** (VAEP evaluated at the relevant events, replacing a frequency ratio) and a **sensitivity analysis** showing ball-gain prediction saturates at three or four observed players.

**[[drso|DRSO]]** abandons that paradigm entirely. No classifiers, **no machine learning at all** — a physical value surface ([[obso|OBSO]]) plus a search over candidate positions. It asks a different question: not *how well did the defence perform* but **where should each defender have stood**.

So the line runs classifier → classifier-with-better-weights → **counterfactual search**, and the third step is a change of kind rather than degree.

## The Contribution That Mattered Most Here

DRSO computes $Diff_{opt-obs}$ **for each named defender** — the vault's first held framework to do so from collective spatial data. It closes the mechanism half of a gap this vault carried across six log entries.

It does not close the reporting half: every published result averages three defenders, then events, then teams. Nothing in the method prevents player-level output; the authors simply did not produce it. See [[defensive-valuation]].

## Two Design Choices Worth Borrowing

**Restriction as an accommodation to bad data.** [[drso|EF-OBSO]] computes only for attacking-third events, justified on three independent grounds — more players visible (especially the keeper), movement direction becomes assumable without velocity data, and it is where the question lives. A principled restriction rather than a convenience.

**Interpretability chosen over capability.** The authors argue explicitly that ML's low interpretability limits practical application, and that a physical model gives coaches more usable advice. Rare: interpretability is normally traded *against* something here, not preferred outright. See [[theory-based-modelling]] and [[interpretability]].

## A Caution Carried Forward

Both papers set PPCF parameters $\sigma = 0.45$, $\lambda = 4.3$ citing [[beyond-expected-goals|Spearman (2018)]], which actually fits $s = 0.54$, $\lambda = 3.99$. See [[obso]] — a citation error propagating through the line, invisible without the primary source.

## See Also

- [[gvdep]] · [[drso]] · [[defensive-valuation]] · [[obso]] · [[counterfactual-baseline]] · [[off-ball-value]]
- [[vdep]] · [[model-selection]] · [[theory-based-modelling]] · [[pitch-control]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[kosuke-toda]] · [[masakiyo-teranishi]] · [[nagoya-university]]
- [[generalized-vdep-euro-location-analysis|GVDEP Summary]] · [[team-defense-positioning-counterfactuals|DRSO Summary]]
