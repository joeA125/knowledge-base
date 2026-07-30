---
title: "Multi-Object Tracking"
type: concept
tags: [multi-object-tracking, computer-vision, object-detection, optical-tracking-data, sports-analytics, metric-learning, evaluation]
sources: [raw/papers/soccernet-game-state-reconstruction.md, raw/papers/detection-tracking-football-broadcast-footage.md, raw/papers/computer-vision-football-review.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Multi-Object Tracking

Following several objects across video frames while maintaining **consistent identities**. Detection asks *what is in this frame*; tracking asks *which of these is the same thing as before*.

## Tracking-by-Detection

The dominant paradigm, and a two-stage one:

1. **Detect** objects independently in each frame — see [[object-detection]].
2. **Associate** detections across frames into tracks.

Association combines motion prediction (a Kalman filter or similar, giving where each track should appear next) with appearance matching (an embedding comparing crops; see [[siamese-network]] and [[metric-learning]]). Assignment is usually solved per frame by the Hungarian algorithm on a cost matrix of predicted-versus-observed pairs.

The two cues are complementary in a specific way: motion is reliable over short gaps and useless after an occlusion; appearance survives occlusion and fails when objects look alike.

## Why Football Is Hard

The sport is close to a worst case, and the reasons compound:

- **Uniform appearance.** Teammates wear identical kits, so the appearance cue — the one that ought to survive occlusion — is nearly useless *within* a team. This is the defining difficulty, and it has no equivalent in pedestrian tracking.
- **Frequent occlusion.** Players cluster at set pieces and in duels, precisely when tracking matters most.
- **Erratic motion.** Sprints, sudden changes of direction, and physical contact break constant-velocity assumptions.
- **Moving camera.** Broadcast footage pans and zooms, so image motion mixes object motion with camera motion. Disentangling them needs [[camera-calibration]] or [[image-alignment|frame registration]].
- **Small, low-resolution targets** in wide shots.

The consequence is **identity switches** — the characteristic failure, where two tracks swap. A switch does not merely lose a player; it corrupts both trajectories, which is why identity-aware metrics matter more than detection accuracy here.

## Where It Sits in the Pipeline

Tracking is the step that turns video into the data everything else in this vault consumes:

$$\text{video} \to \text{detection} \to \text{tracking} \to \text{[[camera-calibration]]} \to \text{[[optical-tracking-data]]}$$

[[game-state-reconstruction]] is the end-to-end version of this pipeline — detect, track, identify by jersey number and team, and localise onto pitch coordinates.

The dependency is worth making explicit because it is easy to forget how much rests on it. [[pitch-control]], [[obso|OBSO]], [[c-obso]], [[soccermap]] and every tracking-based valuation framework take player positions as *given*. They are not given; they are the output of a tracker, with identity switches and localisation error baked in, and **no vault source propagates tracking uncertainty into downstream value estimates.**

## Evaluation

Detection metrics are insufficient because they ignore identity. The standard measures:

- **MOTA** — combines false positives, misses and identity switches into one number; dominated by detection quality.
- **IDF1** — identity-aware F1 over the whole track, so it penalises switches properly.
- **HOTA** — explicitly balances detection and association, which MOTA does not.

The commercial systems producing the vault's tracking data ([[stats-perform]], [[data-stadium]]) report none of these, so the error characteristics of the position data underlying every tracking-based finding here are **unknown**.

## See Also

- [[object-detection]] · [[game-state-reconstruction]] · [[camera-calibration]] · [[image-alignment]]
- [[optical-tracking-data]] · [[siamese-network]] · [[metric-learning]] · [[optical-flow]]
- [[jaccard-index]] · [[stats-perform]] · [[data-stadium]]
- [[soccernet-game-state-reconstruction|SoccerNet GSR Summary]] · [[detection-tracking-football-broadcast-footage|Detection and Tracking Summary]] · [[computer-vision-football-review|CV Review]]
