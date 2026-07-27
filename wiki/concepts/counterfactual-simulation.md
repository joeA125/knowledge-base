---
title: "Counterfactual Simulation"
type: concept
tags: [counterfactual, generative-model, sports-analytics, player-evaluation, evaluation, machine-learning, entity-embedding, transfer-prediction, recruitment]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/eventgpt-player-impact-from-team-action-sequences.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-27
---

# Counterfactual Simulation

Counterfactual simulation uses a generative model to answer "what would have happened if...?" — generating outcomes under conditions that were never observed. It differs from prediction in that the conditioning is *hypothetical*: not "what happens next given this situation", but "what would happen in a situation that did not occur".

## Why Valuation Is Not Enough

The [[action-valuation]] frameworks in this vault — [[vaep]], [[expected-threat|xT]], [[on-ball-value|OBV]], [[martingale-epv]] — all value actions that **actually happened**. That answers "how good was this player's contribution?" but not "how good would this player be *for us*?"

The objection is put sharply in both papers from this line. A transfer is not a like-for-like substitution: moving a player changes the tactical configuration and reshapes interaction patterns, so past performance is drawn from a different distribution than future performance will be. The question is behaviour **under distribution shift**.

[[eventgpt-player-impact-team-action-sequences|Lee, Hong et al.]] add a second objection aimed at the value models specifically — that value is "applied as a post-hoc layer on completed event sequences… rather than co-learned with the sequential process that generates actions." Their answer is [[on-ball-value|residual OBV]], a forward-looking value target predicted *inside* the generative process.

## Two Strengths of Counterfactual

An important distinction, and the axis on which this line of work advanced:

| | **Re-scoring** | **Re-generation** |
|---|---|---|
| Procedure | Hold the observed sequence fixed, substitute the player, re-evaluate value | Substitute the player, generate a new sequence |
| Question answered | How would this player *value* these situations? | How would this player *change what happens*? |
| Example | [[eventgpt]] | [[scoutgpt]] |
| Exposure to [[teacher-forcing\|compounding error]] | None — nothing generated | Substantial over long rollouts |

Re-scoring is the weaker counterfactual but the safer estimate. Re-generation asks the question you actually care about and pays for it in accumulated generation error. Neither paper measures that cost directly.

## What a Model Needs to Support It

1. **Generative.** It must produce sequences, not score existing ones — at least for re-generation.
2. **Long enough horizon.** Fragment-level generation forces the remaining value to be approximated; evaluating a transfer needs whole possessions.
3. **Explicit entity conditioning.** The intervention must be *surgical*. If entity identity is entangled with everything else, or generated freely, the counterfactual is not isolated.

The third is achieved in both models by the same trick, established in [[eventgpt]]: **player identity conditions the prediction but is never itself predicted.** Substituting the identity token therefore cannot be silently overruled by the model regenerating the player it expected.

## The Cheaper Alternative: Regression on Context

Simulation is not the only way to ask a counterfactual question, and it is worth being clear about what the expensive machinery actually buys.

[[transfer-performance-prediction|Shelopugin's regression approach]] asks the same question — what will this player produce at that club? — without generating anything. The destination enters as features: [[league-strength-rating|Glicko-2 ratings]] of the target club and league, the difference between old and new league ratings, mean opponent rating, and league style.

| | Generative simulation | Regression on context |
|---|---|---|
| Destination represented as | The actual squad | A strength scalar |
| Captures tactical interaction | **Yes** | No |
| Data required | Full event streams for the destination | Season aggregates + match results |
| Scales to a whole market | No | **Yes** |
| Addresses [[selection-bias\|selection]] in observed transfers | Not addressed | Explicitly, if heuristically |

The comparison is not flattering to either in isolation. Regression cannot tell you that a player suits *these* teammates — its model of a club is a single number, so two clubs of equal strength and different style are interchangeable to it. Simulation can, but needs data most clubs do not hold for most destinations, and — notably — **neither generative paper addresses the selection problem at all**, despite training on exactly the same non-randomly-assigned transfer data.

The practical reading is sequential rather than competitive: regression to narrow a market to a shortlist, simulation to discriminate among the survivors. See [[recruitment]].

## Estimation by Monte Carlo

Generation is stochastic, so a single rollout is a sample rather than an estimate. Counterfactual quantities are computed by averaging over many sampled continuations, and [[scoutgpt]]'s self-to-self reconstruction error falls monotonically with sample count ($1.9 \to 1.5 \times 10^{-3}$ from 1 to 20 samples). **A single rollout is not a counterfactual estimate.**

Aggregation is not always a plain mean. [[eventgpt]] uses a **truncated mean over the top quartile** for attackers, whose value distributions are heavily skewed, and an arithmetic mean elsewhere — defensible given the skew, but a hand-chosen position-dependent estimator that makes roles non-comparable.

## Validation

**Self-to-self reconstruction.** Simulate with the *actual* entity and compare against what really happened. Necessary but not sufficient — a model could reconstruct well while responding wrongly to interventions.

It also fails informatively. EventGPT's simulated rOBV for Saka (18.59) *exceeds* his ground truth (15.72), and the authors then use the simulated value as the comparison baseline. That is an acknowledged but uncorrected bias, and it means their reported gaps are measured against the model's own optimism rather than reality.

**Out-of-sample intervention.** The stronger test: simulate the entity into a genuinely new context and compare against what actually happened there. ScoutGPT's transfer prediction (MAE 1.25 vs 1.88 naive) is this, though against a weak baseline.

The baseline weakness is worth dwelling on, because the regression line offers a calibration point. Shelopugin stratifies his results by whether the player changed club, league, both, or neither — showing that the naive persistence baseline degrades sharply for movers (0.050 → 0.061 RMSE) while his model holds. Neither generative paper stratifies this way, so it is unknown whether their improvements come from genuine context modelling or from the stay-put majority of their evaluation set. **Reporting movers separately should be standard here and currently is not.**

## Sanity Checks Worth Borrowing

EventGPT's case studies include two checks that generalise to any counterfactual system:

- **Does the intervention produce differentiated effects?** Substituting strikers into different tactical contexts *reverses* their ranking — Haaland's predicted value falls from 2.71 in Manchester City's structured build-up to 1.37 in a transition-heavy context. A model that produced the same ranking everywhere would not be modelling context at all.
- **Does it degrade where it should?** Substituting a striker into defensive contexts collapses his projected value. Critically, the model has **no positional labels**, so it cannot be penalising him for being "out of position" — the decline comes purely from contextual demands. This is a genuine falsification test, and it passes.

Shelopugin's model fails an analogous check and says so: it mispredicts when a centre-forward is deployed as a winger, because the target is unconditional on role. That the generative models handle role implicitly while the regression cannot is one of the clearer arguments for the expensive approach.

## Causal Caveats

The language is borrowed from causal inference, and the borrowing is loose. A generative model trained on observational data learns the *observational* distribution; intervening and regenerating gives the correct causal answer only if the model captured the right dependency structure and there is no unmeasured confounding.

In football, observed performance is confounded with team quality, tactical system, and opposition. Conditioning on lineup absorbs some of this, but nothing guarantees the learned association is the causal effect. A further confounder that neither generative paper addresses: the transfers in the training data were **chosen** by clubs forecasting the same quantity, so the observed sample of moves is filtered by scouting judgement. See [[positive-unlabeled-learning]].

Both papers also note **training-window sensitivity**: an unusually poor observed season propagates into the baseline against which substitutions are compared. The transfer results are evidence the simulation is *useful*, not proof it is causally valid.

## Beyond Sport

The pattern generalises wherever a generative model can be conditioned on an intervenable entity: patient outcomes under alternative treatments, user behaviour under different recommendations, system behaviour under configuration changes. The same three requirements apply, and the same caveat about observational training data.

## See Also

- [[eventgpt]] · [[scoutgpt]] · [[on-ball-value]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]]
- [[selection-bias]] · [[positive-unlabeled-learning]]
- [[player-embedding]] · [[teacher-forcing]] · [[action-valuation]]
- [[scoutgpt-counterfactual-player-valuation|ScoutGPT Summary]]
- [[eventgpt-player-impact-team-action-sequences|EventGPT Summary]]
- [[epv-control-duel-skills-football|EPV Control and Duel Summary]]
