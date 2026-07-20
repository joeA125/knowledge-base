---
title: "Event Stream Data"
type: concept
tags: [sports-analytics, data-engineering, event-stream-data]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# Event Stream Data

Event stream data is one of two primary data modalities for analysing soccer games (the other being optical tracking data). It annotates the times, locations, and types of specific events that occur during a game — passes, shots, tackles, cards, and so on.

## Event Stream vs Optical Tracking Data

| Dimension | Event Stream | Optical Tracking |
|---|---|---|
| What it records | Discrete on-ball events (time, location, type) | Continuous positions of all players + ball at high frequency |
| Cost | Cheap, widely available | Expensive (requires camera installations) |
| Availability | Most professional leagues | Wealthy leagues/clubs only |
| Cross-league sharing | Common | Rare |
| Off-ball information | None | Full (all 22 players tracked) |

The [[evaluating-football-player-actions|VAEP paper]] focuses on event stream data because of its wider availability, though the [[vaep]] framework can extend to tracking data. Optical tracking is the modality produced by the computer-vision [[game-state-reconstruction]] pipelines elsewhere in this vault — meaning GSR effectively generates tracking-style data from broadcast video.

## Vendors

Multiple companies produce event stream data, each with proprietary formats: Opta, Wyscout, STATS, Second Spectrum, SciSports, and StatsBomb. This fragmentation is the central data-engineering problem the [[spadl|SPADL language]] was designed to solve.

## Five Data Science Challenges

The VAEP paper identifies why raw event stream data resists analysis:

1. **Mixed objectives:** Data is designed for broadcasters/journalists, not analysts; important fields may be missing.
2. **Inconsistent terminology:** Each vendor uses unique event definitions, so analysis code isn't portable.
3. **Backward-compatibility bloat:** Decade-old formats accumulate suboptimal legacy design choices.
4. **Optional information snippets:** Per-event optional fields make automatic processing hard.
5. **Variable-length features:** Most ML algorithms require fixed-length vectors, but event streams have variable structure.

## Relation to Game State Reconstruction

Event stream data and [[game-state-reconstruction|GSR]] are complementary: event streams provide rich semantic annotations (what happened) but no continuous positional data, while GSR reconstructs positional/tracking data from video but must infer event semantics. A complete analytics system combines both.

## See Also

- [[spadl]]
- [[vaep]]
- [[game-state-reconstruction]]
- [[evaluating-football-player-actions|Source Summary]]
