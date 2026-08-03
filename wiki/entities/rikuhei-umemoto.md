---
title: "Rikuhei Umemoto"
type: entity
tags: [person, researcher, university, sports-analytics, defensive-valuation, off-ball]
sources: [raw/papers/defensive_player_location_analysis.md]
confidence: 0.75
provenance:
  extracted: 60%
  inferred: 30%
  generated: 0%
  imported: 0%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Rikuhei Umemoto

Researcher at the Graduate School of Informatics, [[nagoya-university]]. Lead author of [[generalized-vdep-euro-location-analysis|the GVDEP paper]] (2022), with [[kazushi-tsutsui]] and [[keisuke-fujii]].

## Contribution

[[gvdep]] generalises [[vdep|Toda et al.'s VDEP]] by replacing its frequency-derived weighting constant with **score-scaled weights taken from [[vaep|VAEP]]** at the moments ball gains and effective attacks occur. That fixes the single most arbitrary parameter in its predecessor — and does so by deriving it from an existing model rather than asserting a new value, which is a route to parameter choice the rest of this literature has not used. See [[model-selection]].

The second contribution is methodological and more portable: a **sweep over the number of observed players**, showing that ball-gain prediction saturates at three or four while scores, concedes and being-attacked gain nothing from player positions at all. That makes defensive valuation available from broadcast-frame data rather than a tracking licence.

## The 2023 Work, Not Held

Umemoto is also credited with **Umemoto & Fujii (2023)**, *Evaluation of team defense positioning by computing counterfactuals using StatsBomb 360 data* (StatsBomb Conference) — the counterfactual-positioning method that would individuate defensive credit.

**That is a different paper and is not held here.** The vault previously conflated the two, treating the Umemoto line as having closed the individual-defender gap. GVDEP is **team-level**; the gap remains open, and the 2023 work is the vault's outstanding acquisition target for it. See [[defensive-valuation]].

## Position in the Fujii Group

The [[keisuke-fujii|group's]] defensive line runs Toda → Umemoto → Umemoto, each addressing the previous paper's stated limitation:

| Work | Fixes |
|---|---|
| [[vdep\|Toda et al. (2022)]] | Establishes proxy-target defensive valuation |
| **[[gvdep\|Umemoto et al. (2022)]]** | **The arbitrary weight; the full-observation assumption; single-league scope** |
| Umemoto & Fujii (2023), unheld | Individual defender credit, by counterfactual positioning |

**Note:** vault knowledge of this person comes from one primary source and one citation. Nothing beyond these two works is established.

## See Also

- [[gvdep]] · [[vdep]] · [[defensive-valuation]] · [[model-selection]]
- [[keisuke-fujii]] · [[kazushi-tsutsui]] · [[kosuke-toda]] · [[masakiyo-teranishi]] · [[nagoya-university]]
- [[generalized-vdep-euro-location-analysis|Source Summary]]
