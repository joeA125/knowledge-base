---
title: "Theory-Based Modelling"
type: concept
tags: [theory-based-modelling, feature-engineering, machine-learning, statistics, interpretability, sports-analytics, model-decomposition, representation-learning]
sources: [raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/expected_value_possession_framework.md, raw/papers/beyond_expected_goals.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 45%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Theory-Based Modelling

Encoding domain knowledge — physics, geometry, or explicit statistical structure — as a **model** rather than as features, and often using that model's *output* as an input to a learned one.

Distinct from [[feature-engineering]], and the distinction is the point. A hand-crafted feature is a transformation of the inputs. A theory-based model is a **self-contained estimator with its own parameters and its own semantics**, whose prediction happens to be useful downstream.

## The Hybrid Pattern

The recurring architecture:

$$\text{raw inputs} \longrightarrow \underbrace{\text{theory-based model}}_{\text{explicit structure}} \longrightarrow \hat{p} \longrightarrow \underbrace{\text{learned model}}_{\text{residual structure}} \longrightarrow \text{prediction}$$

The theory model captures what is known; the learned model captures what is not. Two independent instances in this vault:

| Source | Theory component | Fed into |
|---|---|---|
| [[optimal-decisions-shot-taking-situations\|Yeung & Fujii]] | Geometric shot-block model (truncated normal per defender, integrated over shot angle) | $MLP_{block}$ |
| [[expected-value-possession-framework\|Fernández et al.]] | Event-data [[expected-goals\|xG]] on 118k shots | Tracking-data xG on 14k shots |

Two groups arriving at the same architectural idea independently is worth noting.

## The Ablation That Justifies It

Yeung & Fujii report the cleanest evidence, on shot-block prediction (cross-entropy, lower better):

| Configuration | CEL |
|---|---|
| **MLP + theory-based feature** | **0.4876** |
| MLP, basic shooter features only | 0.5545 |
| **MLP, raw player coordinates (22 × 4)** | **0.5684** |
| Theory-based model alone | 0.9220 |

Three things fall out.

**The hybrid beats both components.** Theory alone is much worse than the MLP; the MLP alone is worse than the pair. Neither ingredient is sufficient.

**Raw coordinates are worse than nothing.** Handing the network a 93-dimensional player vector performs *below* using shooter features alone. With 2,575 examples, the information is there but unlearnable.

**The theory model is doing dimensionality reduction the data cannot support learning.** It compresses 22 player positions into one number that means something — and the compression is correct by construction rather than estimated.

## When It Is Worth Doing

The conditions are fairly specific, and worth stating because the pattern is not universally advisable.

**Small data relative to input dimension.** This is the binding one. Both vault instances involve thousands, not millions, of examples. Given 8.5M actions, [[vaep]] learns from raw features successfully.

**Genuine domain structure.** Shot blocking really is geometric — angle, distance, occlusion. Where the structure is real, encoding it is free information; where it is folklore, it is bias.

**The theory model is independently meaningful.** Yeung & Fujii's block model outputs a probability an analyst can inspect and dispute. That is [[interpretability]] in the compositional sense — see [[structured-model-decomposition]].

## The Tension With Learned Representations

This sits directly against the vault's other finding on the same question. [[sig-model]] degrades when handcrafted geometry is added, because a [[path-signature]] recovers that geometry by construction; [[seq2event]] degrades without it. See [[representation-learning]].

The two are reconcilable, and the reconciliation is the useful part:

> **Encode structure the representation cannot recover and the data cannot support learning. Encode nothing else.**

Sig-Model's representation *can* recover path geometry, so adding it is redundant. Yeung & Fujii's MLP *cannot* recover occlusion geometry from 2,575 examples, so adding it is informative. The disagreement is about which regime you are in, not about whether domain knowledge helps.

## Contrast With Purely Theory-Based Models

Some vault models are theory *without* a learned component — [[obso|OBSO]] and its PPCF are physical throughout, with six parameters fitted by MAP. That is the extreme end of the same axis:

| | Purely learned | Hybrid | Purely theory-based |
|---|---|---|---|
| Example | [[vaep]] | Yeung & Fujii, Fernández et al. | [[obso]], [[pitch-control\|PPCF]] |
| Data needed | Large | Moderate | **Small** |
| Parameters | Many, uninterpretable | Mixed | **Few, with units** |
| Captures the unmodelled | **Yes** | Yes | No |

The far-right column buys an underrated advantage: parameters with **physical units** admit priors from previous measurement, which is how [[beyond-expected-goals|Spearman]] fits six parameters on five matches. See [[model-selection]].

## See Also

- [[feature-engineering]] · [[representation-learning]] · [[structured-model-decomposition]] · [[interpretability]]
- [[xsot]] · [[obso]] · [[pitch-control]] · [[c-obso]] · [[expected-goals]]
- [[model-selection]] · [[game-theory]]
- [[optimal-decisions-shot-taking-situations|Source Summary]] · [[beyond-expected-goals|Spearman Summary]]
