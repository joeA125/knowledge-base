---
title: "Counterfactual Simulation"
type: concept
tags: [counterfactual, generative-model, sports-analytics, player-evaluation, evaluation, machine-learning]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.85
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Counterfactual Simulation

Counterfactual simulation uses a generative model to answer "what would have happened if...?" — generating outcomes under conditions that were never observed. It differs from prediction in that the conditioning is *hypothetical*: not "what happens next given this situation", but "what would happen in a situation that did not occur".

## Why Valuation Is Not Enough

The [[action-valuation]] frameworks in this vault — [[vaep]], [[expected-threat|xT]], [[martingale-epv]] — all value actions that **actually happened**. That answers "how good was this player's contribution?" but not "how good would this player be *for us*?"

[[scoutgpt-counterfactual-player-valuation|Hong et al. (2026)]] put the objection sharply: a transfer is not a like-for-like substitution. Moving a player changes the tactical configuration and reshapes interaction patterns across the whole team, so past performance is drawn from a different distribution than future performance will be. The question is behaviour **under distribution shift**, which extrapolation cannot supply.

Forecasting models ([[seq2event]], [[nmstpp]], [[sig-model]]) come closer — they generate — but are trained and evaluated on predicting *observed* continuations, and generally lack the entity-conditioning needed to hold everything fixed while changing one thing.

## What a Model Needs to Support It

Three requirements, and the third is the one usually missing:

1. **Generative.** It must produce sequences, not score existing ones.
2. **Long enough horizon.** Fragment-level generation forces the remaining value to be approximated; evaluating a transfer needs whole possessions.
3. **Explicit entity conditioning.** The intervention must be *surgical* — change the player, hold the context fixed. If player identity is entangled with everything else, or generated freely, the counterfactual is not isolated.

[[scoutgpt]] achieves the third by conditioning on an explicit lineup context block and **never generating player identity** — it is resolved deterministically from the lineup given the predicted team and position. Swapping the lineup therefore swaps the player without the model being able to overrule the intervention.

## Estimation by Monte Carlo

Generation is stochastic, so a single rollout is a sample rather than an estimate. Counterfactual quantities are computed by averaging over many sampled continuations.

The stability matters empirically: ScoutGPT's self-to-self reconstruction error falls monotonically as samples increase (per-episode mean $1.9 \to 1.5 \times 10^{-3}$ from 1 to 20 samples). **A single rollout is not a counterfactual estimate**, and reported results should state the sample count.

## Validation: the Self-to-Self Check

Counterfactual claims are hard to validate because the counterfactual world is unobserved. One available check is **self-to-self reconstruction**: simulate with the *actual* lineup and compare against what really happened. If the model cannot reproduce the factual, its counterfactuals are not credible.

This is necessary but not sufficient — a model could reconstruct well while responding wrongly to interventions. The stronger test used here is out-of-sample transfer prediction: simulate a player into their new team and compare against their actual subsequent season (MAE 1.25 vs 1.88 for naive carry-over).

## Causal Caveats

The language of counterfactuals is borrowed from causal inference, and the borrowing is loose. A generative model trained on observational data learns the *observational* distribution; intervening on one variable and regenerating gives the correct causal answer only if the model has captured the right dependency structure and there is no unmeasured confounding.

In football, a player's observed performance is confounded with their team's quality, tactical system, and opposition. A model conditioned on lineup may absorb some of this, but nothing guarantees the learned association is the causal effect. The transfer-prediction results are evidence the simulation is *useful*, not proof it is causally valid.

The vault's other causal-flavoured work — the [[martingale|martingale]] requirement in [[martingale-epv]] — takes a different route to the same worry, insisting on stochastic consistency so that changes in the value curve reflect what players did rather than estimator artifacts.

## Beyond Sport

The pattern generalises to any domain where a generative model can be conditioned on an intervenable entity: simulating patient outcomes under alternative treatments, user behaviour under different recommendations, or system behaviour under configuration changes. The same three requirements apply, and the same caveat about observational training data.

## See Also

- [[scoutgpt]]
- [[large-event-model]]
- [[action-valuation]]
- [[vaep]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
