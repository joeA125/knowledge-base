---
title: "Model-Based Clustering of Football Event Sequences: A Marked Spatio-Temporal Point Process Mixture Approach — Source Summary"
type: source_summary
tags: [sports-analytics, clustering, mixture-model, point-process, expectation-maximization, markov-model, event-stream-data, spatiotemporal, statistics, identifiability]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Model-Based Clustering of Football Event Sequences

**Authors:** [[koffi-amezouwui]], [[brigitte-gelein]], [[matthieu-marbac]], [[anthony-sorel]]
**Affiliations:** Univ. Rennes / Ensai / CREST; IRMAR; Université Bretagne Sud / LMBA; Inria / M2S
**Published:** 2025

## Key Contribution

A [[mixture-model]] whose components are marked spatio-temporal [[point-process]]es, enabling **clustering of entire possessions** rather than modelling or valuing individual events. Each component combines three sub-models:

| Component | Model |
|---|---|
| Event types (marks) | Finite [[absorbing-markov-chain|Markov chain with an absorbing state]] for ball loss |
| Event times | Conditional Gamma process (captures over- and under-dispersion) |
| Event locations | Truncated Brownian motion on the pitch |

Estimated by maximum likelihood via a [[expectation-maximization|Generalised EM]] algorithm. Applied to StatsBomb data from 38 Ligue 1 matches (2020/21), clustering the 2,623 possessions faced by Stade Rennais in defence.

## A Different Statistical Unit, and a Different Goal

This is the vault's third point-process treatment of football events, and it differs from both predecessors on two axes:

| | [[nmstpp]] / [[sig-model]] | **This paper** |
|---|---|---|
| Statistical unit | The event (predict the next one) | The **possession** (cluster whole sequences) |
| Goal | Forecasting, then valuation | Unsupervised structure discovery |
| Output | Next action's time/location/type | Latent cluster membership |

The stated motivation is unusual and concrete: populating **virtual reality training environments** for Rennes goalkeepers with realistic, tactically homogeneous opponent possessions. If a possession in a real match produced a bad outcome, its cluster can be identified and other possessions from that cluster used to build rehearsal scenarios.

## The Model

For possession $i$ with cluster indicator $\mathbf{Z}_i$, the marginal density is

$$f(\boldsymbol{x}_i; \boldsymbol{\theta}) = \sum_{k=1}^K \pi_k \, \underbrace{g(\boldsymbol{s}_i; \boldsymbol{\Gamma}_k)}_{\text{marks}} \, \underbrace{h(\boldsymbol{t}_i \mid \boldsymbol{s}_i; \boldsymbol{\rho}_k)}_{\text{times}} \, \underbrace{\ell(\boldsymbol{u}_i \mid \boldsymbol{s}_i, \boldsymbol{t}_i; \boldsymbol{\eta}_k)}_{\text{locations}}$$

This is the same chain-rule factorisation used by [[nmstpp]], but nested inside a mixture — each cluster gets its own transition matrix, timing parameters, and spatial variances.

**Marks.** A finite Markov chain over 16 event types, with *End Possession* as an absorbing state. Possession length is therefore a random absorption time rather than a modelling choice.

**Times.** A conditional Gamma process, with inter-event times conditionally independent given event type and cluster. Gamma rather than exponential because football event data show **no self-excitation** (following Narayanan et al., 2023) but do show dispersion — the observed mean inter-event time is 2.49s with variance 41.06, heavily over-dispersed.

**Locations.** Each coordinate follows a truncated Gaussian centred at the previous location with variance $\eta^2_{k,h,e}\Delta t$ — which reduces to **two-dimensional truncated Brownian motion** when variances are held constant across event types. Truncation keeps events on the $[0,120] \times [0,80]$ pitch.

## Identifiability

[[identifiability|Lemma 1]] establishes that parameters are identifiable up to label swapping under three conditions: positive mixing proportions, a transition matrix making $\{1,\dots,E\}$ a single aperiodic transient communication class, and **distinct Gamma parameters across components**.

The proof works by taking $\Delta t \to \infty$ and exploiting the tail behaviour of the Gamma densities to peel off components one at a time. That the Gamma parameters must differ across clusters is the substantive requirement — clusters distinguished only by their spatial or transition structure would not be separable by this argument.

## Interpretable Summary Statistics

Rather than reporting raw transition matrices, the paper derives three quantities from them using [[absorbing-markov-chain|absorbing-chain theory]] with fundamental matrix $\boldsymbol{F}_k = (\boldsymbol{I}_E - \boldsymbol{Q}_k)^{-1}$:

- **Expected events per possession:** $\lambda_k = 1 + \boldsymbol{a}_k^\top \boldsymbol{F}_k \boldsymbol{1}_E$
- **Expected visits per event type:** $\boldsymbol{\kappa}_k = \boldsymbol{a}_k^\top \boldsymbol{F}_k$
- **Expected possession duration:** $\zeta_k = \boldsymbol{\mu}_k^\top [\boldsymbol{\kappa}_k, 1]^\top$

This turns a $16 \times 16$ transition matrix per cluster into three interpretable numbers a coach can read.

## Estimation

[[expectation-maximization|GEM]] rather than full EM, because the M-step has no closed form for the Gamma and truncated-Gaussian parameters. Closed-form updates exist for $\boldsymbol{\pi}$ and $\boldsymbol{\Gamma}_k$; the rest use a **single L-BFGS-B iteration** per GEM step, which still guarantees the likelihood increases.

Multi-start strategy: 1,000 random initialisations, top 100 after 10 iterations refined for 500 more, run on 80 CPU cores. Total ≈3.3 hours for $K \in \{1,\dots,6\}$.

## Results

$K = 4$ selected by BIC (−264,121 against −264,383 for $K=3$ and −264,141 for $K=5$).

| | Cluster 1 | Cluster 2 | Cluster 3 | Cluster 4 |
|---|---|---|---|---|
| Proportion | 0.37 | 0.31 | 0.13 | 0.19 |
| Events $\lambda_k$ | 5.60 | 11.56 | 11.01 | 15.04 |
| Duration $\zeta_k$ (s) | 25.88 | 28.41 | 19.65 | 33.13 |
| Passes | 2.21 | 4.89 | 4.05 | 6.62 |
| Carries | 0.98 | 3.71 | 3.85 | 5.01 |

The interpretation maps onto recognised tactical vocabulary: Cluster 1 is short direct **counter-attacking**; Cluster 4 is elaborate **positional play / tiki-taka**; Clusters 2 and 3 are intermediate. The authors read this as evidence for a *continuum* of tactical approaches rather than discrete categories.

Note that Cluster 3 has nearly as many events as Cluster 2 (11.0 vs 11.6) in substantially less time (19.7s vs 28.4s) — a distinct high-tempo contested profile that event counts alone would miss. The joint temporal modelling is what separates them.

### Match outcomes
Winning teams show more Cluster 1 (0.38 vs 0.33 in defeats) and Cluster 4; defeats associate with Cluster 2 dominance. Away teams favour Cluster 1 (direct transitions), home teams Clusters 2 and 4 (positional play).

The authors are careful not to overclaim: Cluster 4 dominance accompanies a Reims draw, a Dijon win, and an Angers defeat — possession elaboration alone does not determine outcome.

## Simulation Study

ARI ([[adjusted-rand-index]]) against known ground truth across three separation levels and $n \in \{50,100,200,400\}$:

| Scenario | $n=50$ | $n=400$ |
|---|---|---|
| Easy | 0.642 | 0.833 |
| Intermediate | 0.444 | 0.698 |
| Hard | 0.356 | 0.524 |

Recovery improves with sample size, consistent with maximum-likelihood consistency — but note that even the easy case plateaus around 0.83 rather than approaching 1.

## Relation to Narayanan et al. (2023)

[[sandeep-narayanan|Narayanan, Kosmidis & Dellaportas]] is the closest prior work, and §3.4 draws the contrast explicitly:

| | Narayanan et al. | This paper |
|---|---|---|
| Statistical unit | Whole match | Possession |
| Mixture / clustering | No | **Yes** |
| Spatial representation | 3 discretised zones | Continuous coordinates |
| Transition law | Rich parameterisation | Simpler (less data per unit) |

Both adopt the Gamma process for inter-event times on the same grounds — football events do not self-excite, so a [[point-process|Hawkes process]] would be the wrong choice.

## Limitations

- Single team's defensive possessions from one season; generalisation untested.
- ~4% of possessions discarded for labelling errors, plus exclusions for implausible velocity.
- Spatial model is Brownian — a driftless random walk. Real possessions have directional intent, which a drift term would capture.
- Inter-event times are conditionally independent given event type, so the model cannot represent rhythm or tempo changes within a possession.

## See Also

- [[mixture-model]]
- [[expectation-maximization]]
- [[absorbing-markov-chain]]
- [[identifiability]]
- [[adjusted-rand-index]]
- [[point-process]]
