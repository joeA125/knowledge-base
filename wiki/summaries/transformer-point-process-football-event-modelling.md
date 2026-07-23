---
title: "Transformer-Based Neural Marked Spatio Temporal Point Process Model for Football Match Events Analysis — Source Summary"
type: source_summary
tags: [sports-analytics, point-process, event-prediction, transformer, deep-learning, event-stream-data, player-evaluation, spatiotemporal, stochastic-process]
sources: [raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Transformer-Based Neural Marked Spatio Temporal Point Process Model for Football Match Events Analysis

**Authors:** [[calvin-yeung]], [[tony-sit]], [[keisuke-fujii]]
**Affiliations:** Nagoya University; Chinese University of Hong Kong; RIKEN; JST PRESTO
**Published:** 2023

## Key Contribution

Two contributions. First, [[nmstpp]] — a model that treats football [[event-stream-data]] as a marked spatio-temporal [[point-process]] and forecasts the **time, location, and action type** of the next event *jointly and dependently*, using a [[transformer]] encoder for history representation under the [[neural-temporal-point-process|NTPP]] framework. Second, [[hpus]] — a possession-evaluation metric derived from the model's forecasts.

## The Gap It Addresses

Prior sports sequence models forecast only *part* of the next event — usually location and action type — and model those components independently. The paper argues all three factors must be modelled together:

> Shots are frequently taken when players are close to the opponent's goalpost, but rarely when far away. Furthermore, if it takes a long time before the team gets the ball close to the opponent's goal, the opponent will have sufficient time to react, with shot-taking no longer being the best option.

Temporal information in particular has been overlooked. [[expected-threat|xT]] discards time entirely; [[vaep]] includes some temporal features but does not forecast timing.

## Formulation as a Point Process

A marked spatio-temporal point process $\{(t_i, z_i, m_i)\}$ maps onto football events directly: $t$ is inter-event time, $z$ is the pitch zone (the "spatial" component), $m$ is the action type (the "mark"). By the chain rule (Cox, 1975), the joint conditional density factorises:

$$\prod_i f(t_i, z_i, m_i \mid H_i) = \prod_i f_t(t_i \mid H_i) \, f_z(z_i \mid t_i, H_i) \, f_m(m_i \mid t_i, z_i, H_i)$$

This is what makes the components *dependent without being independent* — each conditional may be modelled separately, but each conditions on the previously predicted components. The ordering is interchangeable in principle, and the paper grid-searches it.

## Architecture (Five Stages)

1. **Input** — 40 previous events (`seqlen = 40`), each with inter-event time, zone (1 of 20), action (1 of 5), plus engineered features (distance/angle to goal, change in zone coordinates).
2. **History encoding** — dense layer on continuous features, embeddings for zone and action, then [[positional-encoding]] and a [[transformer]] encoder producing a fixed-size history vector.
3. **Forecasting** — three neural networks chained per the factorisation: $NN_t(H) \to t$, then $NN_z(t, H) \to \vec{z}$, then $NN_m(t, \vec{z}, H) \to \vec{m}$.
4. **Output** — one value for time, 20 logits for zone, 5 logits for action.
5. **Cost** — $L = \sum 10 \times \text{RMSE}_t + \text{CEL}_z + \text{CEL}_m$, with class-weighted cross-entropy and the ×10 balancing the three terms.

Trained end-to-end with Adam.

## Feature Engineering for Explainability

Pitch $(x,y)$ coordinates are grouped into **20 zones via *Juego de posición*** — the positional-play scheme used by Pep Guardiola and Bayern Munich. Action types are collapsed from WyScout's 21 types / 78 subtypes into 5 classes (pass 67.0%, possession end 19.6%, dribble 8.5%, cross 3.3%, shot 1.7%).

An ablation shows zones perform *identically* to raw $(x,y)$ coordinates (RMSE$_t$ 0.10, CEL$_\text{action}$ 1.33 for both). So the discretisation costs nothing in accuracy while producing output a coach can act on — a rare case where [[interpretability]] is free.

## Results

### Against baselines (validation set)
| Model | Total loss | RMSE$_t$ | CEL$_\text{zone}$ | CEL$_\text{action}$ | Train (min) | Params (K) |
|---|---|---|---|---|---|---|
| AR(2) + transition probs | 6.98 | 0.12 | 2.34 | 3.40 | — | — |
| Modified Seq2Event (Transformer) | 4.57 | 0.11 | 2.11 | 1.39 | 47 | 13 |
| Modified Seq2Event (Uni-LSTM) | 4.51 | 0.10 | 2.11 | 1.37 | 129 | 4 |
| Fine-tuned Seq2Event (Transformer) | 4.48 | 0.10 | 2.09 | 1.36 | 79 | 137 |
| **NMSTPP** | **4.40** | 0.10 | **2.04** | **1.33** | 49 | 79 |

The Transformer-vs-LSTM comparison is notable: the LSTM variant is marginally better on loss but takes **2.7× longer to train** (129 vs 47 min) — consistent with [[seq2event|Simpson et al.'s]] finding that the transformer encoder is slightly less effective but significantly more efficient.

### Ablation: dependence matters
| | Total loss | CEL$_\text{action}$ |
|---|---|---|
| Independent NMSTPP | 4.44 | 1.37 |
| Dependent NMSTPP | **4.40** | **1.33** |

The entire gain lands on action prediction — unsurprising, since action is last in the chain and therefore conditions on both time and zone.

Ordering matters too: the best order $(t, z, m)$ beats the worst $(z, t, m)$ by 0.18 total loss, with the difference again concentrated in action CEL (1.33 vs 1.44).

### Model verification
- Self-attention weights across the 40-event history lie between 0.01 and 0.06 with no trend, suggesting the window is neither too long nor too short.
- The predicted inter-event time CDF matches the empirical CDF closely — despite the model never assuming a parametric distribution for $t$.

## HPUS

[[hpus]] extends the poss-util metric from [[seq2event]] by using the model's expected zone, action, *and* time. Holistic Action Score:

$$\text{HAS} = \frac{\sqrt{E(\text{Zone} \mid H) \, E(\text{Action} \mid \text{Zone}, H)}}{t}$$

with zone scored 0/5/10 by pitch area and action scored 0 (possession loss) / 5 (dribble, pass) / 10 (cross, shot). Dividing by inter-event time rewards *efficiency* — faster actions give the opponent less time to react.

HPUS aggregates HAS across a possession with exponentially decaying weights $\phi(x) = e^{-0.3(x-1)}$ applied backwards from the final action, so the possession's outcome dominates. The decay constant was chosen to give significant weight to 5–6 events, matching the average possession length of 5.2.

### Validation (2017/18 Premier League)
| | Ranking | Goal | xG | HPUS |
|---|---|---|---|---|
| HPUS | −0.78 | 0.92 | 0.92 | 1.00 |
| HPUS+ | −0.77 | 0.91 | 0.90 | 0.96 |

HPUS correlates −0.78 with final league position, against −0.84 for goals and −0.81 for [[expected-goals|xG]] — slightly weaker, but **achieved without ever using goal or shot-outcome data**. That is the substantive claim: possession quality can be assessed from event dynamics alone.

Every team's HPUS ratio (HPUS+ / HPUS) sits near 0.30, suggesting a fairly universal conversion rate from created opportunity to actual attack.

## Data

WyScout Open Access Dataset, 2017/18 season, top five European leagues. Train/validation restricted to Bundesliga for compute reasons (73/7 matches); testing across all five leagues (178 matches). The authors note performance would likely improve with the full training set.

## Limitations

- Training and validation used only ~5% of available data.
- Possession-level analysis only; no player-level attribution, unlike [[vaep]] or [[expected-threat|xT]].
- Five action classes discard considerable detail (e.g. defensive actions are absent entirely — only on-ball events of the possessing team are modelled).
- HPUS scoring weights (0/5/10, the ×10 loss balance, the 0.3 decay) are hand-chosen rather than learned.

## See Also

- [[nmstpp]]
- [[hpus]]
- [[point-process]]
- [[neural-temporal-point-process]]
- [[seq2event]]
- [[action-valuation-frameworks-compared]]
