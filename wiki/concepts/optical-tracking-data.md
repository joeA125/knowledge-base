---
title: "Optical Tracking Data"
type: concept
tags: [sports-analytics, optical-tracking-data, computer-vision, spatiotemporal, data-engineering, multi-object-tracking]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.9
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Optical Tracking Data

Optical tracking data records the continuous positions of every player and the ball throughout a game, sampled at high frequency by fixed camera arrays. It is one of the two dominant data modalities in sports analytics, the other being [[event-stream-data]].

## Scale and Provenance

In 2013 the NBA, with STATS LLC, installed optical tracking in all 30 arenas: 2D coordinates for all 10 players and 3D coordinates for the ball at **25 Hz**, yielding over **1 billion space-time observations per season** ([[multiresolution-stochastic-process-nba-possessions|Cervone et al., 2016]]).

## Comparison with Event Stream Data

| Dimension | Optical Tracking | [[event-stream-data|Event Stream]] |
|---|---|---|
| Records | Continuous positions of all 22/10 participants | Discrete on-ball events |
| Frequency | 25 Hz continuous | Event-triggered (~1,500/match) |
| Off-ball information | Complete | None |
| Cost | High — requires camera installation | Low — human/automated annotation |
| Availability | Wealthy leagues and clubs | Most professional leagues |
| Cross-league sharing | Rare | Common |
| Semantics | Must be inferred | Directly annotated |

The two are complementary rather than competing: event streams say *what happened*, tracking says *where everyone was*. Neither alone gives the full picture.

## What It Enables

Analyses impossible from box scores or event streams alone:

- **Off-ball value.** The [[multiresolution-stochastic-process-nba-possessions|EPV model]] shows Ray Allen as one of the most valuable passing options on a possession where he never touches the ball — visible only because his position and defensive coverage are tracked continuously.
- **Defensive structure.** Franks et al. (2015) infer *who is guarding whom* by modelling each defender's position as a linear combination of the basket, the ball, and a guarded offender ($0.62\mathbf{z}^k + 0.11\mathbf{z}_{bask} + 0.27\mathbf{z}_{ball}$).
- **Movement dynamics.** Per-player acceleration fields reveal that Tony Parker attacks the rim from beyond the perimeter while Dwight Howard only does so from inside the paint.
- **Counterfactual valuation.** [[expected-possession-value]] weights actions that *could* have been taken, not just those that were.

## Relation to Game State Reconstruction

The computer-vision [[game-state-reconstruction]] pipelines elsewhere in this vault attempt to *produce* tracking-style data from ordinary broadcast video — detection, [[multi-object-tracking|tracking]], and [[camera-calibration]] combined to place players in pitch coordinates. GSR is effectively an attempt to democratise optical tracking: to obtain from a single broadcast feed what the NBA obtains from dedicated installed camera arrays.

This makes [[camera-calibration]] accuracy the binding constraint. The [[camera-calibration-benchmarking|ProCC paper]] shows homography-based methods disagree with distortion-aware calibration by over 2.5 metres in some pitch regions — an error that would swamp the fine spatial distinctions EPV-style models depend on.

## Limitations

Even at full resolution, tracking data omits information that matters: hand and foot positioning, jump heights, body orientation, and player intent. The EPV authors stress that analyses built on it are best accompanied by game film and expert judgement.

## See Also

- [[event-stream-data]]
- [[expected-possession-value]]
- [[game-state-reconstruction]]
- [[camera-calibration]]
- [[vaep]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
