---
title: "Trajectory Prediction"
type: concept
tags: [trajectory-prediction, deep-learning, graph-neural-network, vae, rnn, sports-analytics, spatiotemporal, generative-model, imitation-learning, counterfactual]
sources: [raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md]
confidence: 0.8
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Trajectory Prediction

Forecasting the future positions of one or more interacting agents. In team sports the agents are players, the interaction is the whole point, and the horizon is seconds rather than milliseconds.

Distinct from [[event-prediction]], which forecasts *what happens next* as a discrete event; this forecasts *where everyone will be* as continuous motion.

## Why Interaction Dominates

A player's next position depends far more on the other twenty-one than on his own momentum. Models treating agents independently fail badly, and the margin is stark — [[creating-scoring-opportunities-trajectory-prediction|Teranishi et al.]] report endpoint error (MAE, metres):

| | 4 s | 6 s | 8 s | 10 s |
|---|---|---|---|---|
| **GVRNN** | **0.608** | **0.867** | 1.701 | 1.606 |
| VRNN | 5.952 | 7.767 | 9.127 | 10.168 |
| RNN+Gauss | 9.101 | 11.396 | 13.312 | 15.327 |

An order of magnitude between the graph-structured model and the same model without graph structure. Note the fair-comparison caveat the authors flag: GVRNN uses centralised optimisation across agents while the baselines optimise per-player, so the gap conflates architecture with training scheme.

## The Architecture Stack

Three ideas compose, each addressing a failure of the last.

**RNN** — a hidden state summarising history. Struggles with multimodal data: averaging over plausible futures produces an implausible one.

**VRNN** (Chung et al., 2015) — a [[variational-autoencoder|VAE]] conditioned on an RNN hidden state, injecting a stochastic latent $z_t$ at each timestep:

$$p_\theta(z_t | x_{<t}, z_{<t}) = \varphi_{\text{prior}}(h_{t-1}), \qquad h_t = f(x_t, z_t, h_{t-1})$$

Trained by maximising a sequential ELBO — a sum of per-timestep VAE bounds. The latent lets the model represent *several* plausible continuations rather than their mean, and makes it [[generative-model|generative]]: sampling gives distinct trajectories.

**GVRNN** (Yeh et al., 2019) — replaces the VRNN's per-agent networks with [[graph-neural-network|graph neural networks]]. Prior, encoder and decoder all become GNNs over the agent graph:

$$[\mu^{\text{pri}}_{t,1:K}, \sigma^{\text{pri}}_{t,1:K}] = \text{GNN}_{\text{pri}}(h_{t-1,1:K})$$

Each agent's latent distribution is conditioned on every other agent's state through message passing, and the whole model is permutation-equivariant.

## Horizon and Agent Count Are the Binding Constraints

Error grows with prediction length. Teranishi et al. choose **4 seconds** — long enough for movement to differentiate, short enough for sub-metre error.

The 8 s and 10 s figures are not significantly different, which matters: beyond some horizon the model has stopped tracking the actual future and settled into generic plausible motion. That ceiling, rather than raw accuracy, limits downstream use — and it is the same compounding-error problem that [[teacher-forcing|exposure bias]] describes in autoregressive generation.

A second constraint is **agent count**. Teranishi et al. predict only three players — one off-ball attacker and two nearest defenders — because error grows with the number predicted. Fujii has described the full-22 version as prohibitively expensive. So a method whose whole appeal is modelling interaction is in practice restricted to the smallest interacting subset.

## Prediction as a Reference, Not a Forecast

The most interesting use here is not forecasting at all.

[[c-obso]] trains its predictor on *opponent* data to generate "league average" movement, then treats the prediction as a **[[counterfactual-baseline|counterfactual reference]]**: what would a typical player have done? The difference between actual and predicted movement, propagated through [[obso|OBSO]], becomes the player's credit for [[space-creation|creating space]] a teammate exploits.

This inverts the usual objective. A forecaster wants error minimised; a baseline wants a well-calibrated notion of *normal*. C-OBSO is identically zero under perfect prediction, so the two goals are in genuine tension.

The lineage is [[imitation-learning]] rather than forecasting: "ghosting" (Le et al., 2017) trains models to reproduce how a *league-average defence* would have moved, for coaches to compare against. Same idea, applied to defensive shape rather than individual credit.

## Elsewhere

Pedestrian and vehicle trajectory prediction share the machinery and most of the problems — multimodality, interaction, horizon decay — and the sports literature borrows freely from both. The distinguishing feature of the sports case is that **the interaction is adversarial and the agent count is fixed**, which makes graph structure unusually well-suited: the node set never changes and every edge is meaningful.

## See Also

- [[c-obso]] · [[counterfactual-baseline]] · [[imitation-learning]] · [[space-creation]]
- [[graph-neural-network]] · [[variational-autoencoder]] · [[generative-model]] · [[teacher-forcing]]
- [[recurrence]] · [[lstm]] · [[optical-tracking-data]] · [[event-prediction]]
- [[nmstpp]] · [[sig-model]] · [[scoutgpt]] · [[off-ball-value]] · [[pitch-control]]
- [[creating-scoring-opportunities-trajectory-prediction|Source Summary]]
