---
title: "Point Process"
type: concept
tags: [statistics, point-process, stochastic-process, spatiotemporal, density-estimation, sequence-modelling, mixture-model]
sources: [raw/papers/transformer-point-process-football-event-modelling.md, raw/papers/multiresolution-stochastic-process-model-nba-possessions.md, raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 32%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-24
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

For football event data this maps neatly: $t$ = inter-event time, $z$ = pitch location, $m$ = action type.

The joint conditional density can be factorised without assuming independence:

$$f(t_i, z_i, m_i \mid H_i) = f_t(t_i \mid H_i) \, f_z(z_i \mid t_i, H_i) \, f_m(m_i \mid t_i, z_i, H_i)$$

Each component may be modelled by a different mechanism, yet the components remain dependent because each conditions on those before it. The ordering is interchangeable in principle — though empirically it matters, with $(t, z, m)$ outperforming $(z, t, m)$ by 0.18 total loss in [[transformer-point-process-football-event-modelling|Yeung et al.]]

## Classical Families

| Process | Behaviour |
|---|---|
| **Poisson** | Events independent; intensity does not depend on history |
| **Hawkes** | *Self-exciting* — each event raises the intensity of future events, capturing clustering/contagion |
| **Gamma** | Inter-event times gamma-distributed; captures over- and under-dispersion **without** self-excitation |
| **Log-Gaussian Cox** | Intensity is itself a random field with a [[gaussian-process]] log-intensity |

Log-Gaussian Cox processes have been used for basketball shot locations (Miller, Bornn, Adams & Goldsberry, 2014) — the same factorised-intensity work whose [[non-negative-matrix-factorization|NMF]] spatial bases feed into [[martingale-epv]].

## Football Does Not Self-Excite

A substantive empirical constraint, established by [[sandeep-narayanan|Narayanan, Kosmidis & Dellaportas (2023)]] and inherited by subsequent work: **football event data show no self-excitation.** One event does not raise the intensity of subsequent events in the way a Hawkes process assumes.

The data does show strong *dispersion* — [[football-event-sequences-point-process-mixture|Amezouwui et al.]] report mean inter-event time 2.49s against variance 41.06, heavily over-dispersed relative to a Poisson process.

Hence the **Gamma process** has become the standard choice in this literature: more general than Poisson, able to represent both over- and under-dispersion, but without the auto-excitation that would misrepresent the dynamics. Both Narayanan et al. and Amezouwui et al. adopt it on exactly these grounds.

This is a useful example of a domain finding constraining model choice — the intuitive appeal of Hawkes processes for "one thing triggers another" turns out not to survive contact with the data.

## Why Machine Learning Replaced Parametric Specification

Classical point processes require choosing a functional form for the intensity — exponential decay for Hawkes, a gamma distribution for waiting times. This is restrictive and often wrong.

[[neural-temporal-point-process|Neural temporal point processes]] instead learn the conditional distributions from data. The payoff is visible in the [[nmstpp]] results: the predicted inter-event-time CDF closely matches the empirical CDF **without any parametric distribution being assumed** for $t$.

## Mixtures of Point Processes

[[football-event-sequences-point-process-mixture|Amezouwui et al. (2025)]] take a further step: rather than fitting one point process to all data, they fit a [[mixture-model|finite mixture]] whose *components are themselves point processes*.

This shifts the statistical unit from the event to the **realisation** — a whole possession — and enables clustering of entire sequences. It is a different task from forecasting: rather than "what happens next", the question is "what kind of sequence is this".

The three point-process treatments of football in this vault therefore differ in unit and purpose:

| | Unit | Task |
|---|---|---|
| Narayanan et al. (2023) | Match | Modelling, simulation |
| [[nmstpp]] (2023) | Event | Forecasting |
| Amezouwui et al. (2025) | Possession | Clustering |

## Relation to Other Vault Concepts

- **[[competing-risks]]** is a marked point process viewed through its cause-specific intensities. [[martingale-epv|The basketball EPV model]] uses exactly this framing for its macrotransition hazards.
- **[[absorbing-markov-chain|Absorbing Markov chains]]** provide a discrete-state route to the same termination structure — an absorbing state ends the process just as a terminal event does.
- **[[autoregressive-model|Autoregressive models]]** share the chain-rule factorisation, but over a fixed sequence index rather than continuous time. A point process must additionally model *when*.

## See Also

- [[neural-temporal-point-process]]
- [[nmstpp]]
- [[mixture-model]]
- [[competing-risks]]
- [[absorbing-markov-chain]]
- [[sandeep-narayanan]]
- [[football-event-sequences-point-process-mixture|Mixture Source Summary]]
