---
title: "Event Stream Data"
type: concept
tags: [sports-analytics, data-engineering, event-stream-data, optical-tracking-data, event-prediction, clustering]
sources: [raw/papers/evaluating-football-player-actions.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-20
updated: 2026-07-24
---

# Event Stream Data

Event stream data is one of two primary data modalities for analysing team sports (the other being [[optical-tracking-data]]). It annotates the times, locations, and types of specific events that occur during a game — passes, shots, tackles, cards, and so on.

## Event Stream vs Optical Tracking Data

| Dimension | Event Stream | [[optical-tracking-data|Optical Tracking]] |
|---|---|---|
| What it records | Discrete on-ball events (time, location, type) | Continuous positions of all players + ball at high frequency |
| Sampling | Event-triggered | 25 Hz continuous (NBA) |
| Cost | Cheap, widely available | Expensive (requires camera installations) |
| Availability | Most professional leagues | Wealthy leagues/clubs only |
| Cross-league sharing | Common | Rare |
| Off-ball information | None | Full (all players tracked) |
| Semantics | Directly annotated | Must be inferred |
| Collection | Manual video annotation | Automated optical systems |

The [[evaluating-football-player-actions|VAEP paper]] focuses on event stream data because of its wider availability. [[martingale-epv|The basketball EPV model]] takes the opposite bet — building on optical tracking to capture off-ball value that event streams cannot see at all.

Cross-league sharing is the practical reason event streams dominate recruitment work: tracking data is rarely shared between leagues, so a club scouting abroad usually has only event data to work from.

## Modelling Consequences of the Choice

The modality shapes what kind of model is possible:

- **Event streams** are naturally discrete, so [[vaep]] can use fixed-length feature vectors and off-the-shelf [[gradient-boosting]] classifiers, and [[expected-threat|xT]] can solve a small transition matrix by [[value-iteration]].
- **Tracking data** is continuous, which is why [[martingale-epv]] needs a [[multiresolution-modelling|multiresolution]] [[stochastic-process|stochastic process]] model — and why it costs 461 processors to fit.
- A third route treats the event stream *itself* as a continuous-time [[point-process]] ([[nmstpp]], [[football-event-sequences-point-process-mixture|Amezouwui et al.]]), recovering temporal structure that discrete-sequence models discard without needing tracking data.

## Vendors

Multiple companies produce event stream data, each with proprietary formats: Opta, Wyscout, STATS, Second Spectrum, SciSports, and StatsBomb. This fragmentation is the central data-engineering problem the [[spadl|SPADL language]] was designed to solve.

The **WyScout Open Access Dataset** (Pappalardo et al., 2019) is the largest public football event dataset, released specifically to enable research; it underpins [[nmstpp]], [[sig-model]], and much subsequent academic work. **StatsBomb** data is also widely used and offers finer event granularity.

## How Much to Discretise?

Raw vendor taxonomies are far more granular than most models use, and the amount of collapsing varies enormously by task:

| Representation | Action classes | Rationale |
|---|---|---|
| Raw WyScout | 21 types, 78 subtypes | Vendor completeness |
| [[spadl]] | 21 types | Cross-vendor unification, fixed 9-attribute schema |
| [[football-event-sequences-point-process-mixture\|Mixture model]] | 16 types (StatsBomb) | Transition matrix must be estimable per cluster |
| [[sig-model]] | 7 classes | Comparability with prior work |
| [[nmstpp]] | 5 classes | Tractability + [[interpretability]] for coaches |

The pattern is that **forecasting models collapse hardest, structural models least**. A model predicting the next action needs enough examples per class to learn a distribution, and the classes are severely imbalanced — shots are just 1.7% of WyScout events. A model estimating a transition matrix can tolerate more states, since it is characterising structure rather than predicting a label.

The same tension appears spatially, and here the evidence is reassuring: [[nmstpp]] groups $(x,y)$ into 20 zones via *Juego de posición* and finds performance **identical** to raw coordinates, while [[football-event-sequences-point-process-mixture|Amezouwui et al.]] retain continuous coordinates specifically to get finer spatial characterisation than Narayanan et al.'s three zones allowed. Neither choice is simply better — it depends whether spatial precision is an input to prediction or an output to be interpreted.

## Five Data Science Challenges

The VAEP paper identifies why raw event stream data resists analysis:

1. **Mixed objectives:** Data is designed for broadcasters/journalists, not analysts; important fields may be missing.
2. **Inconsistent terminology:** Each vendor uses unique event definitions, so analysis code isn't portable.
3. **Backward-compatibility bloat:** Decade-old formats accumulate suboptimal legacy design choices.
4. **Optional information snippets:** Per-event optional fields make automatic processing hard.
5. **Variable-length features:** Most ML algorithms require fixed-length vectors, but event streams have variable structure.

Manual annotation also introduces errors requiring cleaning: Amezouwui et al. discarded ~4% of possessions for labelling errors, plus further exclusions where implied player velocity exceeded the 98th percentile.

## Relation to Game State Reconstruction

Event stream data and [[game-state-reconstruction|GSR]] are complementary: event streams provide rich semantic annotations (what happened) but no continuous positional data, while GSR reconstructs [[optical-tracking-data|tracking-style data]] from video but must infer event semantics. A complete analytics system combines both.

## See Also

- [[optical-tracking-data]]
- [[spadl]]
- [[vaep]]
- [[expected-threat]]
- [[nmstpp]]
- [[sig-model]]
- [[point-process]]
- [[game-state-reconstruction]]
