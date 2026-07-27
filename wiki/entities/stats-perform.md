---
title: "Stats Perform (STATS LLC / Opta)"
type: entity
tags: [entity, organisation, data-provider, sports-analytics, event-stream-data, optical-tracking-data]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.65
provenance:
  extracted: 35%
  inferred: 50%
  ambiguous: 15%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Stats Perform (STATS LLC / Opta)

Sports data provider supplying both [[optical-tracking-data|tracking]] and [[event-stream-data|event]] data to clubs, broadcasters and researchers. Appears in the vault under several names reflecting a corporate lineage: **STATS LLC** and **Opta** were separate businesses that combined into **Stats Perform**.

## Appearances in This Vault

| Source | Data used |
|---|---|
| [[expected-value-possession-framework\|Fernández, Bornn & Cervone]] | STATS LLC tracking, 633 Premier League matches at 10 Hz |
| Same, baseline xG | Opta event data, 117,948 shots across 4,679 matches |
| [[epv-control-duel-skills-football\|Shelopugin]] | Provider unnamed; Stats Perform published [[garry-gelade\|Gelade's]] 1v1 metric |

The first two are the same paper drawing on both halves of the eventual merger — a useful illustration of why the distinction matters practically. The tracking dataset supports rich spatial modelling but covers two seasons of one league; the event dataset is nearly nine times larger in shots and supports a better-calibrated baseline xG. **The paper uses each where its strength lies**, feeding the event-based xG into the tracking-based models as a prior feature.

## Why Providers Deserve Tracking

Data provenance is not incidental in this field, and the vault's sources are inconsistent about reporting it.

**Annotation conventions are load-bearing.** [[duel-skill-rating]] depends on a definition of who "won" a duel that is partly an artefact of how the provider labels contested events. [[spadl]] exists precisely because vendors describe the same action differently — Wyscout records a duel as two separate events where others record one.

**Availability determines what is studied.** Tracking data exists for wealthy leagues, so tracking-based findings are findings about elite football. This is the data-availability instance of [[selection-bias]] noted across the vault.

**Comparability across papers is limited.** Two frameworks trained on different providers' event streams are not straightforwardly comparable even on identical metrics, because the underlying event taxonomies differ.

A source that does not name its provider — as Shelopugin's does not — is therefore harder to situate than one that does.

## See Also

- [[event-stream-data]] · [[optical-tracking-data]] · [[spadl]]
- [[expected-goals]] · [[duel-skill-rating]] · [[selection-bias]]
- [[garry-gelade]]
- [[expected-value-possession-framework|Source Summary]] · [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
