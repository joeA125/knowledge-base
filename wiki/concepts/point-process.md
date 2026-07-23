---
title: "Point Process"
type: concept
tags: [statistics, point-process, stochastic-process, spatiotemporal, density-estimation, sequence-modelling]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 32%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Point Process

A point process is a probabilistic model for the occurrence of discrete events scattered in time (or space, or both). Where a time series records a value at fixed intervals, a point process models *when the events themselves happen* — the timing is the random quantity, not a sampling grid.

## Two Equivalent Characterisations

### Conditional intensity function
The rate at which events occur, given the history $H_t$:

$$\lambda(t \mid H_t) = \lim_{\Delta t \downarrow 0} \frac{\mathbb{P}(\text{event in } [t, t + \Delta t))}{\Delta t}$$

This is the object most classical point processes specify directly. It is the same construct as the hazard in [[competing-risks]] — a hazard *is* a cause-specific conditional intensity.

### Conditional density of inter-event times
Equivalently, specify $f(t_i \mid H_i)$, the density of the waiting time until the next event. The joint density then factorises by the chain rule:

$$f(t_1, t_2, \dots) = \prod_i f(t_i \mid H_i)$$

[[nmstpp]] takes this route, which suits machine-learning estimation better than intensity specification.

## Marks and Space

- A **marked** point process attaches a label $m_i$ to each event (the "mark") — an event type or category.
- A **spatio-temporal** point process attaches a location $z_i$.
- A **marked spatio-temporal point process (MSTPP)** has both: $\{(t_i, z_i, m_i)\}$.

For football event data this maps neatly: $t$ = inter-event time, $z$ = pitch zone, $m$ = action type (pass, shot, cross, dribble, possession end).

The joint conditional density can be factorised without assuming independence:

$$f(t_i, z_i, m_i \mid H_i) = f_t(t_i \mid H_i) \, f_z(z_i \mid t_i, H_i) \, f_m(m_i \mid t_i, z_i, H_i)$$

Each component may be modelled by a different mechanism, yet the components remain dependent because each conditions on those before it. The ordering is interchangeable in principle — though empirically it matters, with $(t, z, m)$ outperforming $(z, t, m)$ by 0.18 total loss in [[transformer-point-process-football-event-modelling|Yeung et al.]].

## Classical Families

| Process | Behaviour |
|---|---|
| **Poisson** | Events independent; intensity does not depend on history |
| **Hawkes** | *Self-exciting* — each event raises the intensity of future events, capturing clustering/contagion |
| **Log-Gaussian Cox** | Intensity is itself a random field with a [[gaussian-process]] log-intensity |
| **Reactive** | Intensity responds to both events and interventions |

Hawkes processes are the natural fit for football and were used for action types in Narayanan's (2020) football model, since one event genuinely triggers others. Log-Gaussian Cox processes have been used for basketball shot locations (Miller, Bornn, Adams & Goldsberry, 2014) — the same factorised-intensity work whose [[non-negative-matrix-factorization|NMF]] spatial bases feed into [[martingale-epv]].

## Why Machine Learning Replaced Parametric Specification

Classical point processes require choosing a functional form for the intensity — exponential decay for Hawkes, a gamma distribution for waiting times, and so on. This is restrictive and often wrong.

[[neural-temporal-point-process|Neural temporal point processes]] instead learn the conditional distributions from data. The payoff is visible in the [[transformer-point-process-football-event-modelling|NMSTPP results]]: the predicted inter-event-time CDF closely matches the empirical CDF **without any parametric distribution being assumed** for $t$.

## Relation to Other Vault Concepts

- **[[competing-risks]]** is a marked point process viewed through its cause-specific intensities. [[martingale-epv|The basketball EPV model]] uses exactly this framing for its macrotransition hazards.
- **[[markov-game|Markov and semi-Markov processes]]** differ in emphasis: they describe *state* evolution, while point processes describe *event* occurrence. A semi-Markov process is essentially a marked point process whose marks are states.
- **[[autoregressive-model|Autoregressive models]]** share the chain-rule factorisation, but over a fixed sequence index rather than continuous time. A point process must additionally model *when*, which is precisely the extra component NMSTPP adds over [[expected-threat|xT]] and [[vaep]].

## See Also

- [[neural-temporal-point-process]]
- [[nmstpp]]
- [[competing-risks]]
- [[gaussian-process]]
- [[markov-game]]
- [[transformer-point-process-football-event-modelling|Source Summary]]
