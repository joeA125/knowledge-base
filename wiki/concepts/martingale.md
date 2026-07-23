---
title: "Martingale"
type: concept
tags: [statistics, stochastic-process, probabilistic-classification, evaluation]
sources: [raw/papers/multiresolution-stochastic-process-model-nba-possessions.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# Martingale

A martingale is a stochastic process whose expected future value, given everything known so far, equals its current value:

$$\mathbb{E}[M_{t+s} \mid \mathcal{F}_t] = M_t \quad \text{for all } s > 0$$

Informally: it is a "fair game" — no strategy using only past information can produce an expected gain. The filtration $\mathcal{F}_t$ formalises "everything known by time $t$".

## Why Conditional Expectations Are Martingales

Any process defined as a conditional expectation of a fixed terminal quantity, $M_t = \mathbb{E}[X \mid \mathcal{F}_t]$, is automatically a martingale by the tower property:

$$\mathbb{E}[M_{t+s} \mid \mathcal{F}_t] = \mathbb{E}\big[\mathbb{E}[X \mid \mathcal{F}_{t+s}] \mid \mathcal{F}_t\big] = \mathbb{E}[X \mid \mathcal{F}_t] = M_t$$

This is exactly the structure of [[martingale-epv]] and of asset prices under risk-neutral valuation — hence the "stock ticker" reading of EPV.

## Why It Matters for Sports Valuation

[[multiresolution-stochastic-process-nba-possessions|Cervone et al. (2016)]] treat the martingale property as a design requirement, not a curiosity, for two reasons.

### 1. It licenses causal-looking interpretation
If the EPV curve is a martingale, every movement in it is attributable to information arriving — that is, to what players did. A curve produced by a marginal regression on game-state features has no such guarantee: it might jump because the model is miscalibrated in some region of feature space, and an analyst could not tell the two apart. The authors call this *stochastic consistency*, and it is their primary argument against regression-based EPV estimators.

### 2. It forces relative baselines for player metrics
The martingale property has a sharp consequence: $\mathbb{E}[\nu_{t+\epsilon} - \nu_t] = 0$. **On average, EPV does not change while any given player holds the ball.** So summing a player's EPV changes over a season yields approximately zero and measures nothing.

This is why [[martingale-epv|EPVA]] must be defined *relative* to a hypothetical league-average player facing the same situations. The baseline is not a stylistic choice — it is mathematically forced by the martingale structure. Any valuation metric built on a conditional-expectation process inherits this constraint.

## Contrast with Other Action Valuation Frameworks

Neither [[vaep]] nor [[expected-threat|xT]] guarantees a martingale structure, and they are the only frameworks in this vault's [[action-valuation]] cluster that do not.

VAEP's action values come from differencing two independently trained [[gradient-boosting|classifiers]] rather than from a single coherent process model, so VAEP values can be summed directly per player without a relative baseline. The trade-off is that VAEP's fluctuations carry no formal consistency guarantee — a cost the authors of VAEP accept in exchange for tractability and portability to cheap [[event-stream-data]].

xT is computed by [[value-iteration]] to a fixed point, which gives internally consistent zone values but no martingale property over the realised sequence of play, since the state representation discards everything except ball location.

## Related Structures

- **Submartingale / supermartingale** — expected future value is at least / at most the current value.
- **Optional stopping theorem** — under regularity conditions, the expectation at a stopping time equals the initial value, which is why $\mathbb{E}[\nu_T] = \nu_0$ across a possession.
- **Doob decomposition** — any adapted process splits into a martingale plus a predictable drift; the drift is where systematic player skill would show up.

## See Also

- [[martingale-epv]]
- [[expected-possession-value]]
- [[multiresolution-modelling]]
- [[vaep]]
- [[expected-threat]]
- [[markov-game]]
- [[multiresolution-stochastic-process-nba-possessions|Source Summary]]
