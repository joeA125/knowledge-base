---
title: "Counterfactual Simulation"
type: concept
tags: [counterfactual, generative-model, sports-analytics, player-evaluation, evaluation, machine-learning, entity-embedding, transfer-prediction, recruitment, reinforcement-learning, multi-agent, action-space]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/eventgpt-player-impact-from-team-action-sequences.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.85
provenance:
  extracted: 58%
  inferred: 34%
  generated: 6%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-08-07
---

# Counterfactual Simulation

Counterfactual simulation uses a generative model to answer "what would have happened if...?" — generating outcomes under conditions that were never observed. It differs from prediction in that the conditioning is *hypothetical*: not "what happens next given this situation", but "what would happen in a situation that did not occur".

## Why Valuation Is Not Enough

The [[action-valuation]] frameworks in this vault — [[vaep]], [[expected-threat|xT]], [[on-ball-value|OBV]], [[martingale-epv]] — all value actions that **actually happened**. That answers "how good was this player's contribution?" but not "how good would this player be *for us*?"

The objection is put sharply in both papers from this line. A transfer is not a like-for-like substitution: moving a player changes the tactical configuration and reshapes interaction patterns, so past performance is drawn from a different distribution than future performance will be. The question is behaviour **under distribution shift**.

[[eventgpt-player-impact-team-action-sequences|Lee, Hong et al.]] add a second objection aimed at the value models specifically — that value is "applied as a post-hoc layer on completed event sequences… rather than co-learned with the sequential process that generates actions." Their answer is [[on-ball-value|residual OBV]], a forward-looking value target predicted *inside* the generative process.

## Three Strengths of Counterfactual

An important distinction, and the axis on which this line of work advanced. **A third was added 2026-08-07** on ingest of [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]].

| | **Re-scoring** | **Re-generation** | **Value-function readout** |
|---|---|---|---|
| Procedure | Hold the observed sequence fixed, substitute the player, re-evaluate value | Substitute the player, generate a new sequence | Read $Q(s, a')$ for an action $a'$ nobody took |
| Question answered | How would this player *value* these situations? | How would this player *change what happens*? | **How good would this *action* have been?** |
| Example | [[eventgpt]] | [[scoutgpt]] | Nakahara et al. |
| Intervenes on | The **entity** | The **entity** | **The action** |
| Generates anything | No | Yes | **No** |
| Exposure to [[teacher-forcing\|compounding error]] | None | Substantial over long rollouts | **None** |

Re-scoring is the weaker counterfactual but the safer estimate. Re-generation asks the question you actually care about and pays for it in accumulated generation error.

**The third route is different in kind**, and worth separating clearly. It requires no generation at all: a learned $Q$ over an explicit [[action-space-design|action space]] is *defined* at every action, so counterfactual values are already in the output tensor. Nothing is simulated and nothing compounds.

### What the third route pays instead

The cost is displaced rather than avoided, and it is easy to miss because nothing visibly goes wrong.

**1. The unchosen values are barely constrained.** On-policy [[temporal-difference-learning|SARSA]] only ever targets $Q(s_t, a_t)$ for the action taken. Everything else is shaped by network smoothness and [[action-supervision]] — that is, by an *assumption* about human competence rather than by data. See [[free-parameters-load-bearing]].

**2. Nothing else responds.** Nakahara et al.'s agents are [[multi-agent-reinforcement-learning|independent]], so a counterfactual run is evaluated with all 21 other players frozen. Re-generation gets this right by construction; value-function readout cannot get it right at all without a world model — which is the thing this route exists to avoid needing.

**3. The counterfactual is only as rich as the action space.** Eight movement octants and no dynamics constraints, so some "available" actions are not physically available from the current velocity.

**The trade is legible:** re-generation buys a responding world at the price of compounding error; readout buys zero error at the price of a frozen world. Neither is free, and no held source compares them.

## What a Model Needs to Support It

For the two generative routes:

1. **Generative.** It must produce sequences, not score existing ones.
2. **Long enough horizon.** Fragment-level generation forces the remaining value to be approximated; evaluating a transfer needs whole possessions.
3. **Explicit entity conditioning.** The intervention must be *surgical*. If entity identity is entangled with everything else, or generated freely, the counterfactual is not isolated.

The third is achieved in both models by the same trick, established in [[eventgpt]]: **player identity conditions the prediction but is never itself predicted.**

For the readout route the requirements are different and lighter: an **explicit, enumerable action space**, and a value function defined over all of it. That is why [[action-space-design]] is the load-bearing choice there — the action space *is* the set of counterfactuals available.

## The Cheaper Alternative: Regression on Context

Simulation is not the only way to ask a counterfactual question.

[[transfer-performance-prediction|Shelopugin's regression approach]] asks the same question — what will this player produce at that club? — without generating anything. The destination enters as features: [[league-strength-rating|Glicko-2 ratings]] of the target club and league, the difference between old and new league ratings, mean opponent rating, and league style.

| | Generative simulation | Regression on context |
|---|---|---|
| Destination represented as | The actual squad | A strength scalar |
| Captures tactical interaction | **Yes** | No |
| Data required | Full event streams for the destination | Season aggregates + match results |
| Scales to a whole market | No | **Yes** |
| Addresses [[selection-bias\|selection]] in observed transfers | Not addressed | Explicitly, if heuristically |

Regression cannot tell you that a player suits *these* teammates. Simulation can, but needs data most clubs do not hold — and **neither generative paper addresses the selection problem at all**, despite training on non-randomly-assigned transfer data.

The practical reading is sequential rather than competitive: regression to narrow a market to a shortlist, simulation to discriminate among the survivors. See [[recruitment]].

## Estimation by Monte Carlo

Generation is stochastic, so a single rollout is a sample rather than an estimate. [[scoutgpt|ScoutGPT's]] self-to-self reconstruction error falls monotonically with sample count ($1.9 \to 1.5 \times 10^{-3}$ from 1 to 20 samples). **A single rollout is not a counterfactual estimate.**

The readout route has no analogue — $Q(s,a')$ is deterministic given the network. That is a genuine convenience and also a warning: **it produces no uncertainty estimate whatsoever.** Monte Carlo variance is at least an honest signal that the counterfactual is uncertain; a single deterministic number carries none. See [[uncertainty-quantification]].

Aggregation is not always a plain mean. [[eventgpt]] uses a **truncated mean over the top quartile** for attackers, and an arithmetic mean elsewhere — defensible given the skew, but a hand-chosen position-dependent estimator that makes roles non-comparable. Nakahara et al. take a plain mean over frames.

## Validation

**Self-to-self reconstruction.** Simulate with the *actual* entity and compare against what really happened. Necessary but not sufficient.

It also fails informatively. EventGPT's simulated rOBV for Saka (18.59) *exceeds* his ground truth (15.72), and the authors then use the simulated value as the comparison baseline. An acknowledged but uncorrected bias.

**Out-of-sample intervention.** The stronger test: simulate the entity into a genuinely new context and compare against what actually happened there. ScoutGPT's transfer prediction (MAE 1.25 vs 1.88 naive) is this, though against a weak baseline.

Shelopugin stratifies results by whether the player changed club, league, both, or neither — showing the naive persistence baseline degrades sharply for movers (0.050 → 0.061 RMSE) while his model holds. Neither generative paper stratifies this way. **Reporting movers separately should be standard here and currently is not.**

**Neither test is available to the readout route.** There is no ground truth for the value of an action that was not taken, and Nakahara et al. say so directly. They fall back on [[construct-validity|correlation with other metrics]], which is why that page exists.

## Sanity Checks Worth Borrowing

EventGPT's case studies include two checks that generalise to any counterfactual system:

- **Does the intervention produce differentiated effects?** Substituting strikers into different tactical contexts *reverses* their ranking — Haaland's predicted value falls from 2.71 in Manchester City's structured build-up to 1.37 in a transition-heavy context.
- **Does it degrade where it should?** Substituting a striker into defensive contexts collapses his projected value. The model has **no positional labels**, so the decline comes purely from contextual demands. A genuine falsification test, and it passes.

Shelopugin's model fails an analogous check and says so: it mispredicts when a centre-forward is deployed as a winger.

**Neither check has been run on the readout route**, and both are cheap there. Does $Q$ for a forward run differ appropriately between a counter-attack and a settled block? Does it collapse for actions that are physically implausible from the current velocity? The second would directly test limitation 3 above.

## Causal Caveats

The language is borrowed from causal inference, and the borrowing is loose. A generative model trained on observational data learns the *observational* distribution; intervening gives the correct causal answer only if the model captured the right dependency structure and there is no unmeasured confounding.

In football, observed performance is confounded with team quality, tactical system, and opposition. A further confounder neither generative paper addresses: the transfers in the training data were **chosen** by clubs forecasting the same quantity. See [[positive-unlabeled-learning]].

The readout route has its own version, and it is sharper. $Q(s,a')$ estimates the value of $a'$ **under the observed policy of everyone else** — including, via [[action-supervision]], a prior that the observed policy is good. So the counterfactual is not "what if he had run left" but "what if he had run left in a world otherwise behaving as it did, evaluated by a function biased toward what people actually do." That is a considerably narrower claim than the notation suggests.

## Beyond Sport

The pattern generalises wherever a model can be conditioned on an intervenable entity or action: patient outcomes under alternative treatments, user behaviour under different recommendations, system behaviour under configuration changes.

The three-route distinction transfers cleanly and is worth carrying: **re-score when the world is fixed and cheap, re-generate when the world responds, read out a value function when the action set is small and enumerable.** The failure mode in the third case is universal — off-policy action values look like estimates and are often assumptions.

## See Also

- [[eventgpt]] · [[scoutgpt]] · [[on-ball-value]] · [[counterfactual-baseline]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[temporal-difference-learning]] · [[reinforcement-learning]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]] · [[c-obso]] · [[drso]]
- [[selection-bias]] · [[positive-unlabeled-learning]] · [[uncertainty-quantification]] · [[construct-validity]]
- [[player-embedding]] · [[teacher-forcing]] · [[action-valuation]] · [[trajectory-prediction]] · [[observed-versus-optimal-decisions]]
- [[scoutgpt-counterfactual-player-valuation|ScoutGPT Summary]] · [[eventgpt-player-impact-team-action-sequences|EventGPT Summary]] · [[epv-control-duel-skills-football|EPV Control and Duel Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
