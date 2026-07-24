---
title: "Geonhee Jo"
type: entity
tags: [person, researcher, ai-research, university]
sources: [raw/papers/eventgpt-player-impact-from-team-action-sequences.md, raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 40%
  ambiguous: 15%
lifecycle: draft
created: 2026-07-24
updated: 2026-07-24
---

# Geonhee Jo

Researcher in the Department of Artificial Intelligence, University of Seoul. Co-author of both [[eventgpt-player-impact-team-action-sequences|EventGPT]] (2025) and [[scoutgpt-counterfactual-player-valuation|ScoutGPT]] (2026), and lead author of the **VERSA** verified event data format (Jo et al., 2026) that ScoutGPT uses for preprocessing.

VERSA applies a formal state-transition model to enforce validity rules and correct anomalies in raw event streams — inserting missing *Pass Received* events, reordering physically impossible sequences. It also supplies the transition validator behind ScoutGPT's [[constrained-decoding]], so the data-quality work and the generation-constraint work are the same artifact used twice.

> **Provenance note:** vault knowledge comes from authorship and citations within the two papers, not from the VERSA paper itself.

## See Also

- [[eventgpt]]
- [[scoutgpt]]
- [[constrained-decoding]]
- [[sang-ki-ko]]
