---
title: "Counterfactual Simulation"
type: concept
tags: [counterfactual, generative-model, sports-analytics, player-evaluation, evaluation, machine-learning, entity-embedding, transfer-prediction, recruitment, reinforcement-learning, multi-agent, action-space, simulator, domain-adaptation]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/eventgpt-player-impact-from-team-action-sequences.md, raw/papers/epv_control_and_duel_skills_football.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.85
provenance:
  extracted: 60%
  inferred: 32%
  generated: 6%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-08-07
---

# Counterfactual Simulation

Counterfactual simulation uses a generative model to answer "what would have happened if...?" — generating outcomes under conditions that were never observed. It differs from prediction in that the conditioning is *hypothetical*.

## Why Valuation Is Not Enough

The [[action-valuation]] frameworks in this vault — [[vaep]], [[expected-threat|xT]], [[on-ball-value|OBV]], [[martingale-epv]] — all value actions that **actually happened**. That answers "how good was this player's contribution?" but not "how good would this player be *for us*?"

A transfer is not a like-for-like substitution: moving a player changes the tactical configuration and reshapes interaction patterns, so past performance is drawn from a different distribution than future performance will be. The question is behaviour **under distribution shift**.

[[eventgpt-player-impact-team-action-sequences|Lee, Hong et al.]] add a second objection — that value is "applied as a post-hoc layer on completed event sequences… rather than co-learned with the sequential process that generates actions." Their answer is [[on-ball-value|residual OBV]], predicted *inside* the generative process.

## Three Strengths of Counterfactual

| | **Re-scoring** | **Re-generation** | **Value-function readout** |
|---|---|---|---|
| Procedure | Hold the observed sequence fixed, substitute the player, re-evaluate | Substitute the player, generate a new sequence | Read $Q(s, a')$ for an action nobody took |
| Question answered | How would this player *value* these situations? | How would this player *change what happens*? | **How good would this *action* have been?** |
| Example | [[eventgpt]] | [[scoutgpt]], [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] | [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] |
| Intervenes on | The **entity** | The **entity** or **the environment** | **The action** |
| Generates anything | No | Yes | **No** |
| Exposure to [[teacher-forcing\|compounding error]] | None | Substantial over long rollouts | **None** |

Re-scoring is the weaker counterfactual but the safer estimate. Re-generation asks the question you actually care about and pays in accumulated generation error.

The third route requires no generation at all: a learned $Q$ over an explicit [[action-space-design|action space]] is *defined* at every action, so counterfactual values are already in the output tensor.

### What the third route pays instead

**1. The unchosen values are barely constrained.** On-policy [[temporal-difference-learning|SARSA]] only targets $Q(s_t,a_t)$ for the action taken. Everything else is shaped by network smoothness and [[action-supervision]] — an *assumption* about human competence rather than data.

**2. Nothing else responds.** Nakahara et al.'s agents are [[multi-agent-reinforcement-learning|independent]], so a counterfactual run is evaluated with all 21 other players frozen.

**3. The counterfactual is only as rich as the action space.** Eight movement octants and no dynamics constraints, so some "available" actions are not available.

**The trade is legible:** re-generation buys a responding world at the price of compounding error; readout buys zero error at the price of a frozen world.

## What Happens When Re-Generation Is Tried Properly

> **Added 2026-08-07** on ingest of [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]], and it is the vault's only *failed* counterfactual simulation.

[[scoutgpt|ScoutGPT]] and [[eventgpt|EventGPT]] re-generate **event sequences** — a discrete, coarse, event-stream representation where a rollout is a handful of tokens. Fujii et al. re-generate **continuous multi-agent movement** in a simulator, at 10 Hz, with all agents acting.

That is re-generation at its most ambitious, and it does not work. Neither their method nor any baseline reproduced demonstrated football behaviour: DQAAS learned to pass and shoot without moving toward goal; plain DQN moved toward goal without passing or shooting. The authors rule out the algorithm as the cause and attribute the failure to **simulator fidelity**.

> ### `regeneration-fidelity-scales-with-representation-coarseness`
> **Re-generation counterfactuals succeed where the representation is coarse enough that the generative model's errors stay inside the discretisation, and fail where it is fine enough that they compound into physically wrong behaviour.**
> ^[generated: no source states this; drawn from the contrast between ScoutGPT/EventGPT's event-token rollouts and Fujii et al.'s continuous multi-agent rollouts. rests-on: source:scoutgpt-transfer-mae, source:fujii-football-reproducibility-failure]

That claim is uncomfortable for the generative papers rather than for Fujii et al. **ScoutGPT's counterfactuals may work partly because event tokens hide the physics** — a generated sequence of {pass, carry, shot} tokens cannot be *physically* impossible in the way a generated trajectory can. Whether the resulting counterfactual is more trustworthy or merely less falsifiable is not established, and no held source tests it.

It also bears on the vault's [[domain-adaptation|Real-to-Sim]] material: the requirement for re-generation is not just a generative model but **a transition model faithful enough to roll out in**, and in continuous football nobody has one.

## What a Model Needs to Support It

For the two generative routes:

1. **Generative.** It must produce sequences, not score existing ones.
2. **Long enough horizon.** Fragment-level generation forces the remaining value to be approximated.
3. **Explicit entity conditioning.** The intervention must be *surgical*.
4. **A transition model that survives the rollout** — added 2026-08-07, and the one Fujii et al. show is not free.

The third is achieved in both generative papers by the same trick: **player identity conditions the prediction but is never itself predicted.**

For the readout route the requirements are lighter: an **explicit, enumerable action space**, and a value function defined over all of it.

## The Cheaper Alternative: Regression on Context

[[transfer-performance-prediction|Shelopugin's regression approach]] asks the same question without generating anything — the destination enters as features: [[league-strength-rating|Glicko-2 ratings]], league-rating difference, mean opponent rating.

| | Generative simulation | Regression on context |
|---|---|---|
| Destination represented as | The actual squad | A strength scalar |
| Captures tactical interaction | **Yes** | No |
| Data required | Full event streams | Season aggregates + results |
| Scales to a whole market | No | **Yes** |
| Addresses [[selection-bias\|selection]] | Not addressed | Explicitly, if heuristically |

The practical reading is sequential rather than competitive: regression to narrow a market, simulation to discriminate among survivors. See [[recruitment]].

## Estimation by Monte Carlo

Generation is stochastic, so one rollout is a sample. [[scoutgpt|ScoutGPT's]] reconstruction error falls monotonically with sample count ($1.9 \to 1.5 \times 10^{-3}$, 1 to 20 samples). **A single rollout is not a counterfactual estimate.**

The readout route has no analogue — $Q(s,a')$ is deterministic given the network. A genuine convenience, and a warning: **it produces no uncertainty estimate whatsoever.** See [[uncertainty-quantification]].

Fujii et al. report **five random seeds** and, on football, zero standard error across all of them — which looks like robustness and more plausibly indicates convergence to a single degenerate solution.

[[eventgpt]] uses a **truncated mean over the top quartile** for attackers and an arithmetic mean elsewhere — a hand-chosen position-dependent estimator that makes roles non-comparable.

## Validation

**Self-to-self reconstruction.** Simulate with the *actual* entity and compare against what really happened. Necessary but not sufficient, and it fails informatively: EventGPT's simulated rOBV for Saka (18.59) *exceeds* ground truth (15.72), and the authors then use the simulated value as the baseline.

Fujii et al.'s [[dynamic-time-warping|DTW]] distance to the demonstration is the same test under a different name, and the honest version: they report it, it does not improve, and they say so.

**Out-of-sample intervention.** ScoutGPT's transfer prediction (MAE 1.25 vs 1.88 naive) is this, against a weak baseline. Shelopugin stratifies by whether the player changed club, league, both, or neither — **reporting movers separately should be standard here and currently is not.**

**Neither test is available to the readout route.** There is no ground truth for the value of an action not taken, and Nakahara et al. say so. They fall back on [[construct-validity|correlation with other metrics]].

## Sanity Checks Worth Borrowing

EventGPT's case studies include two checks that generalise:

- **Does the intervention produce differentiated effects?** Haaland's predicted value falls from 2.71 in Manchester City's structured build-up to 1.37 in a transition-heavy context.
- **Does it degrade where it should?** Substituting a striker into defensive contexts collapses his projected value, with **no positional labels** in the model. A genuine falsification test, and it passes.

Fujii et al. supply a third worth adopting: **induce a controlled domain gap and check the method still works.** Their predator mobility shifts 120% → 110% between source and target — a deliberately small, known perturbation. **No football counterfactual paper in this vault runs an analogous test**, and it is the cheapest way to find out whether a counterfactual system degrades gracefully.

Neither of the first two checks has been run on the readout route, and both are cheap there.

## Causal Caveats

A generative model trained on observational data learns the *observational* distribution; intervening gives the correct causal answer only if the model captured the right dependency structure and there is no unmeasured confounding.

In football, observed performance is confounded with team quality, tactics, and opposition. A further confounder neither generative paper addresses: the transfers in the training data were **chosen** by clubs forecasting the same quantity. See [[positive-unlabeled-learning]].

The readout route has a sharper version. $Q(s,a')$ estimates the value of $a'$ **under the observed policy of everyone else** — including, via [[action-supervision]], a prior that the observed policy is good. So the counterfactual is "what if he had run left in a world otherwise behaving as it did, evaluated by a function biased toward what people actually do." Considerably narrower than the notation suggests.

## Beyond Sport

The three-route distinction transfers cleanly: **re-score when the world is fixed and cheap, re-generate when the world responds, read out a value function when the action set is small and enumerable.**

The failure mode in the third case is universal — off-policy action values look like estimates and are often assumptions. The failure mode in the second is the one Fujii et al. demonstrate: **a rollout is only as good as the transition model underneath it, and in most real-world multi-agent domains nobody has one.**

## See Also

- [[eventgpt]] · [[scoutgpt]] · [[on-ball-value]] · [[counterfactual-baseline]] · [[domain-adaptation]] · [[nfootball]]
- [[multi-agent-reinforcement-learning]] · [[action-supervision]] · [[action-space-design]] · [[temporal-difference-learning]] · [[reinforcement-learning]] · [[dynamic-time-warping]]
- [[transfer-performance-prediction]] · [[league-strength-rating]] · [[recruitment]] · [[c-obso]] · [[drso]]
- [[selection-bias]] · [[positive-unlabeled-learning]] · [[uncertainty-quantification]] · [[construct-validity]] · [[imitation-reward-tradeoff]]
- [[player-embedding]] · [[teacher-forcing]] · [[action-valuation]] · [[trajectory-prediction]] · [[observed-versus-optimal-decisions]]
- [[scoutgpt-counterfactual-player-valuation|ScoutGPT]] · [[eventgpt-player-impact-team-action-sequences|EventGPT]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
