---
title: "Nagoya University"
type: entity
tags: [entity, organisation, university, ai-research, sports-analytics]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 45%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Nagoya University

Japanese research university. Its Graduate School of Informatics is the base of [[keisuke-fujii]], and through him the institutional home of the vault's Japanese sports-analytics line.

## Why It Appears Repeatedly

Two vault sources originate here, and they are more complementary than they first look:

| Source | Approach | Target |
|---|---|---|
| [[football-defence-evaluation-vdep\|VDEP]] (2022) | Event classification with off-ball state | **Defensive** value from frequent proxies |
| [[transformer-point-process-football-event-modelling\|NMSTPP]] (2023) | Transformer point process | Forecasting event time, zone, action |

Both come at football through **prediction rather than valuation**, deriving metrics downstream from a predictive model rather than defining value directly. VDEP predicts recovery and penetration and combines them; NMSTPP predicts the next event and yields [[hpus]] from the forecasts. That is a recognisable methodological signature, and it distinguishes the group from the Leuven ([[vaep]]) and Barcelona ([[expected-value-possession-framework|EPV framework]]) lines.

Fujii holds joint appointments at Nagoya, the RIKEN Center for Advanced Intelligence Project, and JST PRESTO — a structure typical of Japanese research funding, where a university post is combined with national-institute affiliation and a fixed-term project grant.

## See Also

- [[keisuke-fujii]] · [[masakiyo-teranishi]] · [[calvin-yeung]]
- [[vdep]] · [[nmstpp]] · [[hpus]]
- [[kyoto-university]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[transformer-point-process-football-event-modelling|NMSTPP Summary]]
