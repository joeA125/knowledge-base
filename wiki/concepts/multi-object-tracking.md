---
title: "Multi-Object Tracking"
type: concept
tags: [multi-object-tracking, computer-vision, object-detection, optical-tracking-data, sports-analytics, metric-learning, evaluation, uncertainty-quantification]
sources: [raw/papers/soccernet-game-state-reconstruction.md, raw/papers/detection-tracking-football-broadcast-footage.md, raw/papers/computer-vision-football-review.md, raw/papers/defensive_player_location_analysis.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 40%
  generated: 10%
  imported: 0%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Multi-Object Tracking

Following several objects across video frames while maintaining **consistent identities**. Detection asks *what is in this frame*; tracking asks *which of these is the same thing as before*.

## Tracking-by-Detection

1. **Detect** objects independently in each frame — see [[object-detection]].
2. **Associate** detections across frames into tracks.

Association combines motion prediction (a Kalman filter, giving where each track should appear next) with appearance matching (see [[siamese-network]] and [[metric-learning]]). Assignment is usually solved per frame by the Hungarian algorithm.

The two cues are complementary: motion is reliable over short gaps and useless after an occlusion; appearance survives occlusion and fails when objects look alike.

## Why Football Is Hard

- **Uniform appearance.** Teammates wear identical kits, so the appearance cue — the one that ought to survive occlusion — is nearly useless *within* a team. No equivalent in pedestrian tracking.
- **Frequent occlusion.** Players cluster at set pieces and in duels, precisely when tracking matters most.
- **Erratic motion.** Sprints and contact break constant-velocity assumptions.
- **Moving camera.** Broadcast footage pans and zooms; disentangling image from object motion needs [[camera-calibration]] or [[image-alignment|frame registration]].
- **Small, low-resolution targets** in wide shots.

The consequence is **identity switches** — a switch does not merely lose a player, it corrupts both trajectories.

## Where It Sits in the Pipeline

$$\text{video} \to \text{detection} \to \text{tracking} \to \text{[[camera-calibration]]} \to \text{[[optical-tracking-data]]}$$

[[game-state-reconstruction]] is the end-to-end version. [[pitch-control]], [[obso|OBSO]], [[c-obso]], [[soccermap]] and every tracking-based valuation framework take player positions as *given*. They are not given; they are a tracker's output.

> ### `no-tracking-uncertainty-propagation`
>
> **No source in this vault propagates tracking uncertainty into downstream value estimates.**
>
> ^[generated: an absence claim. rests-on: absence:no-held-source-propagates-tracking-error — ⚠️ expires on ingest of any tracking-based valuation paper. **Weakened 2026-07-27** — see below. Also referenced by [[tracking-error-propagation]] and the synthesis.]

> **Weakened, 2026-07-27.** [[generalized-vdep-euro-location-analysis|Umemoto, Tsutsui & Fujii (2022)]] do not propagate *uncertainty*, but they do measure the cost of **incomplete observation** — sweeping the number of visible players from 0 to 11 and reporting F1 at each. That is the closest thing the vault holds to input-quality sensitivity analysis, and it partially answers the question this claim was raised to pose.

Their result is directly relevant: **ball-gain prediction saturates at three or four players; scores, concedes and being-attacked need none beyond the ball.** So for three of four targets, missing players cost nothing measurable — which bounds how much tracking completeness can matter for those predictions.

It does not address **positional error or identity switches**, which are different failure modes from missing observations. A player tracked to the wrong coordinates is worse than a player absent, because the model has no signal that anything is amiss. Structured error also concentrates under occlusion, so it is worst exactly where value concentrates. See [[tracking-error-propagation]].

## Evaluation

- **MOTA** — combines false positives, misses and identity switches; dominated by detection quality.
- **IDF1** — identity-aware F1 over the whole track, so it penalises switches properly.
- **HOTA** — explicitly balances detection and association.

> ### `providers-report-no-tracking-metrics`
>
> **The commercial providers supplying this vault's tracking data report none of MOTA, IDF1 or HOTA.**
>
> ^[generated: an absence claim about what is *published*, not about what exists. rests-on: absence:no-provider-metrics-in-held-sources — ⚠️ expires if a paper reports its provider's error rates. Weaker than it reads: the vault holds a handful of papers using [[stats-perform]] and [[data-stadium]] data, not a survey of provider documentation.]

Note that a **sensitivity curve does not require knowing the true error rate** — perturb clean data across a range, and a provider's actual rate need only be located on the curve. GVDEP's n_nearest sweep is a partial instance of exactly that method, applied to completeness rather than accuracy.

## See Also

- [[tracking-error-propagation]] — the open question on both absence claims
- [[gvdep]] — the partial answer on incomplete observation
- [[object-detection]] · [[game-state-reconstruction]] · [[camera-calibration]] · [[image-alignment]]
- [[optical-tracking-data]] · [[siamese-network]] · [[metric-learning]] · [[optical-flow]] · [[uncertainty-quantification]]
- [[jaccard-index]] · [[stats-perform]] · [[data-stadium]] · [[pitch-control]]
- [[soccernet-game-state-reconstruction|SoccerNet GSR Summary]] · [[detection-tracking-football-broadcast-footage|Detection and Tracking Summary]] · [[computer-vision-football-review|CV Review]]
