---
title: "NMSTPP"
type: concept
tags: [sports-analytics, point-process, event-prediction, transformer, deep-learning, event-stream-data, interpretability, spatiotemporal]
sources: [raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-27
---

# NMSTPP

The Transformer-Based Neural Marked Spatio Temporal Point Process model ([[transformer-point-process-football-event-modelling|Yeung, Sit & Fujii, 2023]]) forecasts the **time, location, and action type** of the next football event jointly and dependently, treating match [[event-stream-data]] as a marked spatio-temporal [[point-process]].

Its distinguishing claim is temporal: previous [[event-prediction|event-forecasting]] models predict *what* happens next and *where*, but not *when*.

## The Dependent Factorisation

By the chain rule, the joint conditional density splits into three terms, each conditioning on the ones before it:

$$f(t_i, z_i, m_i \mid H_i) = f_t(t_i \mid H_i) \, f_z(z_i \mid t_i, H_i) \, f_m(m_i \mid t_i, z_i, H_i)$$

realised as three chained networks:

$$NN_t(H_i) \to t_i, \qquad NN_z(t_i, H_i) \to \vec{z}_i, \qquad NN_m(t_i, \vec{z}_i, H_i) \to \vec{m}_i$$

**This dependence is the model's core empirical claim, and it is validated.** Severing the links so each network sees only $H_i$ raises total loss from 4.40 to 4.44, with the entire degradation falling on action prediction (CEL 1.33 → 1.37) — as expected, since action sits last in the chain and loses the most conditioning.

Ordering also matters: $(t, z, m)$ beats $(z, t, m)$ by 0.18 total loss.

## Architecture

| Stage | Content |
|---|---|
| **Input** | 40 previous events: inter-event time, zone (1–20), action (1–5), engineered geometry |
| **History encoding** | Dense layer on continuous features + embeddings for zone/action → [[positional-encoding]] → [[transformer]] encoder → 31-dim history vector → dense layer |
| **Forecasting** | Three chained NNs per the factorisation above |
| **Output** | 1 value (time), 20 logits (zone), 5 logits (action) |
| **Cost** | $L = \sum 10 \times \text{RMSE}_t + \text{CEL}_z + \text{CEL}_m$ |

79K trainable parameters; 49 minutes training. Cross-entropy terms are class-weighted to handle severe imbalance (shots are only 1.7% of events); the ×10 on RMSE balances the three loss terms onto comparable scales.

## Juego de Posición Zoning

Pitch coordinates are grouped into 20 zones following ***Juego de posición*** — the positional-play framework associated with Pep Guardiola and Bayern Munich — rather than an arbitrary grid.

The ablation is the interesting part: zones perform *identically* to raw $(x,y)$ coordinates (RMSE$_t$ 0.10 and CEL$_\text{action}$ 1.33 for both). The discretisation costs nothing in accuracy while producing outputs expressed in vocabulary coaches already use. [[interpretability]] here is free, which is unusual — the [[expected-threat|xT]] versus [[vaep]] comparison suggests interpretability is normally paid for in accuracy or coverage.

Contrast with xT's zoning, which is a $16 \times 12$ grid chosen for [[value-iteration]] tractability, not for tactical meaning.

## Performance

Best total loss (4.40) against an AR(2)+transition-probability statistical baseline (6.98) and three [[seq2event]] variants (4.48–4.57), while training in 49 minutes.

## Relation to Other Sports Models

NMSTPP is doing something different from the [[action-valuation]] cluster. Where [[vaep]], [[expected-threat|xT]] and [[martingale-epv]] **value** actions that occurred, NMSTPP **forecasts** what will occur next. Valuation follows only downstream, via [[hpus]].

| | Forecasts time? | Forecasts location? | Forecasts action? | Values actions? |
|---|---|---|---|---|
| [[expected-threat\|xT]] | No | Implicitly (zone transitions) | No | Yes |
| [[vaep]] | No | No | No | Yes |
| [[martingale-epv\|EPV]] | Implicitly (hazards) | Yes | Yes | Yes |
| **NMSTPP** | **Yes** | Yes | Yes | Via [[hpus]] |

[[martingale-epv|The basketball EPV model]] is the closest relative — it too forecasts continuously and models event timing via [[competing-risks]] hazards — but it works from [[optical-tracking-data]] rather than cheap event streams, at vastly greater computational cost.

The forecasting route has an advantage worth stating separately: **it needs no outcome labels.** Every event is a training example, so the goal-sparsity problem that cripples direct valuation never arises — which is how [[hpus]] recovers most of xG's signal while using no goal data at any stage. See [[event-prediction]] and [[rare-event-proxy-targets]].

## Limitations

- Trained on ~5% of the available data for compute reasons; the authors expect gains from full training.
- Only the possessing team's on-ball events are modelled — **defensive actions are entirely absent.** The same group later addressed this directly with [[vdep]], which changes the target rather than the architecture.
- Five action classes discard substantial detail.
- **No player-level attribution**; analysis is at possession and team level only. Also later addressed by the same group, via [[c-obso]]'s counterfactual construction.

The last two are worth reading together: both limitations were resolved not by improving this model but by asking a different question of the same data. See [[keisuke-fujii]] for the group's programme.

## See Also

- [[event-prediction]] · [[point-process]] · [[neural-temporal-point-process]] · [[hpus]]
- [[seq2event]] · [[sig-model]] · [[large-event-model]] · [[transformer]]
- [[vdep]] · [[c-obso]] · [[rare-event-proxy-targets]] · [[keisuke-fujii]]
- [[expected-threat]] · [[vaep]] · [[action-valuation]]
- [[transformer-point-process-football-event-modelling|Source Summary]]
