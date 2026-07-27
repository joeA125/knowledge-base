---
title: "FC Barcelona"
type: entity
tags: [entity, organisation, sports-club, sports-analytics]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.7
provenance:
  extracted: 40%
  inferred: 50%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# FC Barcelona

Professional football club; institutional home of [[javier-fernandez]] during the development of the [[expected-value-possession-framework|EPV framework]], where he led the analytics department.

## Role in the Research

More than an affiliation line. The club's **match analysts collaborated directly** on the framework's contextual features — [[dynamic-pressure-lines]], outplayed-player counts, and the line-breaking concepts were built with them rather than derived independently.

The judgement that line-breaking passes are the dominant driver of goal expectation came from those analysts and was subsequently borne out by the EPV-added distributions. That is an unusually clean instance of practitioner intuition being encoded as a feature and then quantitatively confirmed.

## Why the Provenance Matters

The vault has two clubs as entities. [[fc-porto]] appears as a co-author affiliation on [[football-performance-time-series|Valuing Players Over Time]]; Barcelona here.

Both cases carry the same double edge. Club involvement is what makes the work practitioner-grounded — the applications in this framework are recognisably tools a coaching staff would use, not metrics in search of a purpose. It also means the design was shaped by one club's tactical vocabulary. "Pressure lines" and "playing inside the block" are positional-play concepts, native to the tradition Barcelona sits in; a framework built with a counter-attacking club might have encoded transition speed instead.

Nothing about the model is wrong for this. But the **contextual features are not neutral**, and the choice of what to hand-engineer is a tactical commitment as much as a technical one. See [[dynamic-pressure-lines]] on the interpretability-versus-accuracy tension this creates.

## See Also

- [[javier-fernandez]] · [[fc-porto]]
- [[dynamic-pressure-lines]] · [[tactical-analysis]] · [[expected-possession-value]]
- [[expected-value-possession-framework|Source Summary]]
