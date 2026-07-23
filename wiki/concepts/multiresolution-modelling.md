---
title: "Multiresolution Modelling"
type: concept
tags: [statistics, stochastic-process, markov-model, spatiotemporal, approximation, machine-learning]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Multiresolution Modelling

Multiresolution modelling represents the same process at two or more levels of granularity simultaneously, combining them so that each resolution handles the part of the problem it is best suited to. [[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] introduced this approach for estimating [[martingale-epv]] from basketball [[optical-tracking-data]].

## The Problem It Solves

Modelling a continuous-time process at a single resolution forces an unhappy choice:

- **Fully continuous models** preserve all information but make long-horizon expectations intractable — integrating over every possible future path of a 24-second possession at 25 Hz is hopeless.
- **Coarsened [[markov-model|Markov chain]] models** make expectations trivial (linear algebra on a transition matrix) but require discretising away the spatial detail that motivated collecting the data.

The insight is that this is a false dichotomy: the two can be composed.

## The Construction

Let $Z_t$ be the full-resolution state and $C_t = C(Z_t)$ a coarsening into a finite state space. Define $\delta_t$ as the moment the process exits the next "transition" state — in basketball, when the current pass, shot, or turnover completes. Under two assumptions:

- **(A1)** $C$ is marginally semi-Markov;
- **(A2)** *decoupling* — beyond $\delta_t$, all information in the history up to $t$ is summarised by $C_{\delta_t}$;

the target expectation factorises:

$$\nu_t = \sum_{c \in \mathcal{C}} \underbrace{\mathbb{E}[X \mid C_{\delta_t} = c]}_{\text{coarse: Markov algebra}} \cdot \underbrace{\mathbb{P}(C_{\delta_t} = c \mid \mathcal{F}_t^{(Z)})}_{\text{fine: short-horizon forecast}}$$

The coarse factor is computed once from the transition matrix. The fine factor conditions on complete spatial detail but only needs forecasting to the *next* decoupling event — a horizon of a second or two rather than the whole possession.

## Why the Decoupling Assumption Is Plausible

(A2) asserts that a structural transition resets the informational state. In basketball, when a player passes or shoots, all ten players react to that event; what they were doing beforehand stops mattering for what happens after. Given the pass recipient, their court region, and defensive pressure, prior history adds little. The assumption is domain-motivated rather than mathematically convenient.

Its main failure mode is scripted sequences — pre-set plays that deliberately chain actions together, violating the memorylessness the coarsening assumes.

## Rao-Blackwellisation of the Coarse Chain

A neat consequence: transition counts for the coarse chain need not be tallied from observations. Expected counts can instead be computed by integrating the fitted fine-resolution hazards, giving unbiased lower-variance estimates. This matters for rare states — DeAndre Jordan attempted no three-pointers in 2013–14, yet EPV still requires transition probabilities from those states.

## Generality

The authors present basketball as a case study for a broader pattern applicable wherever continuous monitoring data meets discrete outcomes of interest: traffic monitoring, surveillance, digital marketing attribution, and other sports (soccer, hockey). The requirement is a coarsening for which (A1)–(A2) are defensible.

## Contrast with Related Approaches

| Approach | Consistency | Spatial detail | Tractability |
|---|---|---|---|
| Marginal regression | ✗ (no [[martingale]] structure) | ✓ | ✓ |
| Pure Markov chain | ✓ | ✗ (discretised) | ✓ |
| Fully continuous process | ✓ | ✓ | ✗ |
| **Multiresolution** | ✓ | ✓ | ✓ |

The simpler soccer models in the [[expected-possession-value]] family sit in the second row — they buy tractability by discretising the pitch into zones and accept the resulting loss of detail.

## See Also

- [[martingale-epv]]
- [[expected-possession-value]]
- [[markov-game]]
- [[martingale]]
- [[gaussian-process]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
