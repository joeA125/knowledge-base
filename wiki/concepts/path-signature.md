---
title: "Path Signature"
type: concept
tags: [path-signature, statistics, machine-learning, sequence-modelling, spatiotemporal, feature-engineering, dimensionality-reduction, representation-learning]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.85
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Path Signature

The signature of a path is a sequence of iterated integrals that encodes the path's structure in increasing detail. Introduced by Chen (1954) and brought to prominence through Terry Lyons' rough path theory (Lyons, 1998), it has become a principled non-parametric feature map for time series.

## Definition

For a continuous path of finite variation $X : [s,t] \to \mathbb{R}^d$, the signature is the collection of iterated integrals over all multi-indices:

$$S(X)_{s,t} = \left(1, S(X)^1_{s,t}, \dots, S(X)^d_{s,t}, S(X)^{1,1}_{s,t}, S(X)^{1,2}_{s,t}, \dots\right)$$

$$S(X)^{i_1,\dots,i_k}_{s,t} = \int_{s<t_k<t} \cdots \int_{s<t_1<t_2} dX^{i_1}_{t_1} \cdots dX^{i_k}_{t_k}$$

The signature is infinite-dimensional. In practice it is **truncated at order $M$**, keeping multi-indices of length $\le M$. Truncation error decays factorially, $\mathcal{O}(1/M!)$ (Lyons et al., 2007), so low orders capture most of the structure — [[sig-model]] uses $M = 3$.

## Why It Suits Irregular Time Series

Four properties, all directly relevant to messy real-world sequences:

1. **Length-agnostic.** The signature of a path has the same dimension regardless of how many points the path contains. This removes the need for fixed-length windows or padding.
2. **Sampling-frequency-agnostic.** Irregular gaps between observations are handled natively, since the object is a path rather than a regularly-indexed series.
3. **Order-preserving.** Iterated integrals encode the *sequence* and *interaction* of movements. Summary statistics (mean, variance, skewness, kurtosis) discard order entirely — the usual failing of handcrafted [[feature-engineering|features]] on time series.
4. **Essentially injective.** Under appropriate augmentation, the signature determines the path uniquely up to *tree-like equivalence* (Hambly & Lyons, 2010; Boedihardjo et al., 2016) — roughly, up to retracings that cancel themselves out.

## Computation on Discrete Data

Real data is discrete. The path is recovered by linear interpolation, after which **Chen's identity** (Chen, 1958) reduces the integrals to products over line segments:

$$S(X)^{i_1,\dots,i_k}_{t,t+1} = \frac{1}{k!}\prod_{j=1}^{k}\left(X^{i_j}_{t+1} - X^{i_j}_{t}\right)$$

so computing signatures on piecewise-linear paths involves no integration at all.

## Log-Signatures

The signature is redundant. By the **shuffle product property**, every polynomial in signature terms is itself a linear combination of signature terms — for instance $S(X)^{i,i} = \tfrac{1}{2}(S(X)^i)^2$. The full signature therefore stores the same information many times.

The **log-signature**, $\text{LogSig}(X) = \log(S(X))$, strips this redundancy, capturing equivalent information in far fewer terms. It is the form actually used in practice: [[sig-model]] takes an order-3 log-signature of the $(x, y, T)$ path, yielding 55 dimensions from a 61-dimensional signature.

## Augmentations

Raw signatures lose information — notably the path's starting point and any time parameterisation. **Path augmentations** add dimensions to restore this before computing the signature:

- **Time augmentation** appends the timestamp as an extra coordinate, making the parameterisation recoverable.
- **Visibility / invisibility-reset transformation** encodes the starting point.

Only with appropriate augmentation does the uniqueness-up-to-tree-like-equivalence guarantee hold. Morrill et al. (2020) survey the standard choices.

## Applications Beyond Sport

Signatures have been applied across domains with irregular sequential structure: human action and gesture recognition (Yang et al., 2017, 2022; Shi et al., 2025), Alzheimer's diagnosis prediction (Moore et al., 2019), detecting early depressive and manic episodes in bipolar disorder (Kormilitzin et al., 2017), financial modelling and optimal stopping (Cuchiero et al., 2023; Bayer et al., 2023), generative modelling of financial time series with [[variational-autoencoder|VAEs]] (Buehler et al., 2020), and learning neural SDEs by combining log-signatures with [[rnn|RNNs]] (Liao et al., 2019).

## Comparison with Learned Sequence Encoders

Signatures and neural sequence encoders solve the same problem — turning a variable-length sequence into a fixed-size representation — by opposite means:

| | Path signature | [[transformer]] / [[lstm]] encoder |
|---|---|---|
| Representation | Fixed mathematical transform | Learned from data |
| Variable length | Native | Requires padding or fixed windows |
| Irregular sampling | Native | Needs explicit time features |
| Parameters | None | Many |
| Guarantees | Injective up to tree-like equivalence | None |
| Adapts to task | No | Yes |

The signature's lack of learnable parameters is both its strength (no training cost, no overfitting in the encoder) and its limit (it cannot discard task-irrelevant structure). In [[understanding-football-possessions-path-signatures|Hirnschall & Bajons (2025)]] the tradeoff favours signatures: a plain feedforward network on log-signatures beats a transformer benchmark at ~2.5× lower runtime.

## See Also

- [[sig-model]]
- [[feature-engineering]]
- [[transformer]]
- [[neural-temporal-point-process]]
- [[understanding-football-possessions-path-signatures|Source Summary]]
