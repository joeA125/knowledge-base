---
title: "Sandeep Narayanan"
type: entity
tags: [person, researcher, university]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.7
provenance:
  extracted: 35%
  inferred: 45%
  ambiguous: 20%
lifecycle: draft
created: 2026-07-24
updated: 2026-07-24
---

# Sandeep Narayanan

Lead author of "Flexible marked spatio-temporal point processes with applications to event sequences from association football" (Narayanan, Kosmidis & Dellaportas, *JRSS-C*, 2023) — the foundational Bayesian [[point-process]] treatment of football event data.

The work models on-ball actions as a marked spatio-temporal point process at the **match** level, decoupling event times from event types, capturing team-specific abilities and event interactions across locations, and supporting simulation of event sequences.

## Influence Across the Vault

Two later papers here build on it directly, and both adopt its central empirical finding:

- [[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] devote a section to the comparison. They shift the statistical unit from match to **possession** and add a [[mixture-model|mixture]] structure for clustering, which Narayanan's framework does not support. They also retain continuous coordinates where Narayanan discretises the pitch into three zones.
- [[nmstpp|Yeung et al. (2023)]] cite the Hawkes-process element of this line of work when motivating their own formulation.

**The shared finding:** football event data show **no self-excitation** — one event does not raise the intensity of subsequent ones in the way a Hawkes process assumes. This is why both papers use a **Gamma process** for inter-event times, which handles over- and under-dispersion without auto-excitation. It is a genuine empirical constraint on how football event timing may be modelled, established by this work and inherited downstream.

> **Provenance note:** vault knowledge comes only from citations within later papers, not the primary source.

## See Also

- [[point-process]]
- [[mixture-model]]
- [[nmstpp]]
- [[football-event-sequences-point-process-mixture|Source Summary]]
