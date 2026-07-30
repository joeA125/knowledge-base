---
title: "Survival Analysis"
type: concept
tags: [survival-analysis, statistics, stochastic-process, point-process, regression, selection-bias, inference]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.8
provenance:
  extracted: 30%
  inferred: 65%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Survival Analysis

Modelling **time until an event** — and, crucially, handling the cases where the event has not happened yet.

## The Hazard Function

The central object is not a probability but an instantaneous rate:

$$\lambda(t) = \lim_{\Delta t \to 0} \frac{\mathbb{P}(t \le T < t + \Delta t \mid T \ge t)}{\Delta t}$$

"Given survival to $t$, how fast is failure arriving now?" From it follow the survival function $S(t) = \exp(-\int_0^t \lambda(u)\,du)$ and the density.

Working in hazards rather than densities is what makes **censoring** tractable. A unit still alive at the end of observation contributes $S(t)$ to the likelihood rather than being discarded — which is why the framework exists.

## Why It Belongs in This Vault

Football and basketball possessions are time-to-event problems that are rarely framed as such.

[[martingale-epv|Cervone et al.]] model the next macrotransition — pass, shot or turnover — with **cause-specific hazards**, one per event type, competing to fire first. See [[competing-risks]], which is the specific machinery; this page is its parent.

The connection to [[point-process|point processes]] is direct: a point process is characterised by its conditional intensity, which *is* a hazard conditioned on history. [[nmstpp]]'s time component — predicting the inter-event interval — is a hazard model wearing different notation, and [[neural-temporal-point-process|neural TPPs]] are the modern version.

So three vault topics are one topic: **competing risks, point-process intensities, and hazard models.**

## Censoring, and Where It Is Ignored

Censoring is the framework's reason for existing, and the vault's sources mostly sidestep it — possessions in a completed match all end observably.

But the concept recurs unnamed elsewhere, and where it does it is handled badly. [[transfer-performance-prediction|Shelopugin's]] transfer model needs a player to appear next season at all, and handles attrition with a **separate classifier** predicting whether he clears a 100-minute threshold. That is a censoring problem — the outcome is unobserved for reasons correlated with the outcome — approached as a classification problem instead.

A survival framing would be more natural: model *time until a player drops out* with covariates including contract length, and let censored careers contribute properly rather than being filtered or patched. Nothing in the vault does this, and the [[player-development-curve|age-curve]] work has the same shape — its survivorship problem *is* selection on an unmodelled hazard. See [[selection-bias]].

## Standard Machinery, Unused Here

- **Cox proportional hazards** — semiparametric regression on the hazard, leaving the baseline unspecified.
- **Kaplan–Meier** — nonparametric survival curve estimation.
- **Accelerated failure time** — covariates scale time rather than the hazard.

None appears in the vault's sources, which is itself notable given how many of its problems are time-to-event in structure: injury recurrence, career length, time to a team's next goal.

## See Also

- [[competing-risks]] · [[point-process]] · [[neural-temporal-point-process]] · [[stochastic-process]]
- [[martingale-epv]] · [[nmstpp]] · [[absorbing-markov-chain]]
- [[selection-bias]] · [[player-development-curve]] · [[transfer-performance-prediction]]
- [[multiresolution-stochastic-process-nba-possessions|Basketball EPV Summary]]
