---
title: "SPADL (Soccer Player Action Description Language)"
type: concept
tags: [sports-analytics, data-engineering]
sources: [raw/papers/evaluating-football-player-actions.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-20
---

# SPADL (Soccer Player Action Description Language)

SPADL ([[evaluating-football-player-actions|Decroos et al., 2019]]) is a unified language for representing soccer event stream data, designed to address five data science challenges with vendor-specific formats (Opta, Wyscout, StatsBomb): conflicting terminology, backward compatibility bloat, mixed objectives, optional information snippets, and variable-length features.

## Representation

SPADL represents a game as a sequence of on-ball actions $[a_1, a_2, \ldots, a_m]$, where each action is a fixed tuple of 9 attributes:

| Attribute | Description |
|---|---|
| StartTime | Action's start time |
| EndTime | Action's end time |
| StartLoc | $(x, y)$ start location |
| EndLoc | $(x, y)$ end location |
| Player | Player performing the action |
| Team | Player's team |
| ActionType | One of 21 types (pass, shot, dribble, tackle, etc.) |
| BodyPart | foot, head, other, or none |
| Result | success, fail, offside, own goal, yellow card, red card |

The key design principle is **fixed-length representation**: every action has exactly the same 9 attributes, eliminating optional information snippets. This makes actions directly amenable to machine learning algorithms that require fixed-length feature vectors.

## Actions vs Events

SPADL distinguishes between actions (requiring a player to perform them — passes, shots, tackles) and events (which include non-player events like game start/end, referee decisions). Only actions are represented.

## Action Types

21 types covering all on-ball actions: pass, cross, throw-in, crossed corner, short corner, crossed free-kick, short free-kick, take-on, foul, tackle, interception, shot, penalty shot, free-kick shot, keeper save, keeper claim, keeper punch, keeper pick-up, clearance, bad touch, dribble. Passes dominate (64.6%), followed by dribbles (8.7%) and interceptions (5.0%).

## Impact

SPADL provides the data foundation for the [[vaep]] framework and has been adopted by the soccer analytics research community as a common interchange format. A Python package automatically converts Opta, Wyscout, and StatsBomb event streams into SPADL.

## See Also

- [[vaep]]
- [[evaluating-football-player-actions|Source Summary]]
- [[game-state-reconstruction]]
