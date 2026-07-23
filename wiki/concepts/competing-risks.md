---
title: "Competing Risks"
type: concept
tags: [statistics, survival-analysis, stochastic-process, point-process, probabilistic-classification, sports-analytics]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 27%
  ambiguous: 8%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Competing Risks

A competing risks model (Prentice et al., 1978) describes a situation where a subject is at risk of several mutually exclusive event types, and the occurrence of one precludes the others. Rather than modelling *whether* an event happens and then *which* type, it assigns each type $j$ its own cause-specific hazard:

$$\lambda_j(t) = \lim_{\epsilon \to 0} \frac{\mathbb{P}(\text{event } j \text{ occurs in } (t, t+\epsilon] \mid \mathcal{F}_t)}{\epsilon}$$

Because the types are disjoint, the overall hazard is simply $\lambda(t) = \sum_j \lambda_j(t)$, and conditional on an event occurring, the probability it is of type $j$ is $\lambda_j(t) / \sum_k \lambda_k(t)$.

## Relation to Point Processes

A cause-specific hazard **is** a conditional intensity: compare the definition above with the intensity of a [[point-process]],

$$\lambda(t \mid H_t) = \lim_{\Delta t \downarrow 0} \frac{\mathbb{P}(\text{event in } [t, t + \Delta t))}{\Delta t}$$

They are the same object. A competing risks model is therefore a *marked* point process specified through its per-mark intensities, where the marks are the competing event types.

The two literatures emphasise different things: survival analysis focuses on time-to-first-event with censoring, while point processes focus on recurrent events over a whole sequence. But the machinery is shared, which is why football and basketball models can be built in either idiom — [[martingale-epv]] uses competing-risks hazards, while [[nmstpp]] specifies conditional densities directly, and both are modelling the same underlying phenomenon.

## Application to Basketball Macrotransitions

[[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] use competing risks for the *macrotransition entry model* in [[martingale-epv]] — predicting which ball-movement event ends the current possession state. Six competing event types:

| $j$ | Event |
|---|---|
| 1–4 | Pass to each of four teammates (indexed by position) |
| 5 | Shot attempt |
| 6 | Turnover |

Each hazard is log-linear:

$$\log \lambda_j^\ell(t) = [\mathbf{W}_j^\ell(t)]' \boldsymbol{\beta}_j^\ell + \xi_j^\ell(\mathbf{z}^\ell(t)) + \tilde{\xi}_j^\ell(\mathbf{z}_j(t))\mathbf{1}[j \le 4]$$

combining time-varying covariates (distance to nearest defender, whether the player has dribbled, how open a receiver is) with a [[gaussian-process]] spatial effect. For passes there is a second spatial term for the *recipient's* location — capturing that a pass's attractiveness depends on where both players stand.

## Why This Framing Fits

Three properties make competing risks natural here:

1. **Genuine mutual exclusivity.** A player cannot simultaneously pass and shoot; the first event to fire ends the state.
2. **Continuous time.** The tracking data is effectively continuous at 25 Hz, so a hazard formulation is more faithful than discretising into per-frame multinomial choices.
3. **Interpretability as decision-making.** Each hazard describes a player's propensity toward one option as a function of his circumstances, so the fitted parameters read directly as behavioural tendencies. Dwight Howard's three-point hazard is near zero everywhere; Stephen Curry's remains substantial even under close defence.

## Poisson Regression Equivalence

The partial likelihood for the hazards,

$$L_j^\ell \propto \Big(\prod_{t: M_j(t)} \lambda_j^\ell(t)\Big)\exp\Big(-\sum_t \lambda_j^\ell(t)\Big)$$

is exactly a Poisson regression likelihood. This lets the authors fit the whole system with standard machinery ([[inla]]) despite a design matrix of 30.4 million rows and ~6,000 columns.

## Rao-Blackwellising Transition Counts

The fitted hazards do double duty: integrating them gives *expected* transition counts for the coarsened [[markov-game|Markov chain]], replacing raw observed counts. This is unbiased and lower-variance, and crucially gives sensible estimates for state transitions that were never observed.

## Related Concepts

- Competing risks generalise standard survival analysis (a single absorbing event) to multiple absorbing causes.
- The multinomial-logit alternative discards timing information; the hazard formulation retains it.
- Thomas et al. (2013) apply competing-process hazard models to ice hockey player ratings — a direct antecedent in sports analytics.

## See Also

- [[point-process]]
- [[martingale-epv]]
- [[nmstpp]]
- [[multiresolution-modelling]]
- [[gaussian-process]]
- [[inla]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
