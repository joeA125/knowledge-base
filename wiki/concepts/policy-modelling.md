---
title: "Policy Modelling"
type: concept
tags: [policy-modelling, reinforcement-learning, markov-model, sports-analytics, action-valuation, machine-learning, evaluation]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.75
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Policy Modelling

Estimating the policy that agents **actually follow**, rather than solving for the one they should. In an MDP framing, learning $\pi(s,a) = \mathbb{P}(A = a \mid s)$ from observed behaviour and treating it as a quantity of interest in its own right.

## The Inversion

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] frame possession value as a Markov decision process and then state the departure plainly: the aim is not to find the optimal policy $\pi$, but to estimate value **under the average policy learned from historical data**.

This inverts the usual objective. Reinforcement learning normally treats the behaviour policy as a means to an end — something to improve, or to correct for via importance weighting. Here it is the estimand.

Two reasons, and both generalise beyond football.

**The counterfactual would be unfounded.** An optimal-policy value function answers "what is this position worth to a perfect player?" Nobody is a perfect player, and the data contains no examples of perfect play from which to learn one. Value under observed behaviour is answerable from observed behaviour.

**The policy is itself the finding.** Knowing that a player in a given situation passes 29.3% of the time, drives 23.0%, and shoots 47.6% is *directly* useful to a coach — arguably more so than the value estimate. Where behaviour diverges from what the value model rewards, that gap is the coaching intervention.

## Where It Enters the Decomposition

Two policy components appear in the [[structured-model-decomposition|EPV decomposition]], at different granularities:

| Component | Question | Output |
|---|---|---|
| Action selection $\mathbb{P}(A|T_t)$ | Pass, drive, or shoot? | 3-way softmax |
| Pass selection $\mathbb{P}(D_t|A=\rho, T_t)$ | Pass *where*? | [[probability-surface\|Surface summing to 1]] |

Both act as **weights** on the corresponding value estimates. EPV is the value of each option weighted by how likely it is to be taken — an expectation over the behaviour policy, not a maximum over options.

Pass selection is a subtle and easily-missed distinction from pass *success*. Success asks "would this pass work?"; selection asks "would this pass be attempted?" A pass into a dangerous area might have high success probability and near-zero selection probability, because players do not think to play it. That gap is where the framework's most interesting coaching output lives.

## Value Under Policy vs Value Under Optimum

The framework reports both, and the difference is the deliverable. In the worked control-room example, the policy-weighted EPV is 0.032 while the best available pass would yield 0.112.

The first says what this possession is worth given how footballers behave. The second says what it is worth to someone who spots the best option. Neither alone tells a coach anything actionable; **the gap between them does** — it is the value being left on the pitch by ordinary decision-making.

This is the same structure as the advantage function in RL, $A(s,a) = Q(s,a) - V(s)$, repurposed as an interpretive device rather than a training signal.

## Caveats

- **The policy is a population average.** It describes how Premier League players behave, not how *this* player behaves. Conditioning on player identity would give individual policies and is not attempted here — though it is precisely what [[eventgpt]] and [[scoutgpt]] do with player-conditioned generation.
- **Behaviour encodes coaching, not just judgement.** A player may pass backwards because instructed to. The policy conflates individual decision-making with team instruction, and nothing separates them.
- **Circularity risk.** If the value model is trained on outcomes generated under this policy, then "the policy is suboptimal by the value model's lights" is a claim about internal consistency, not about football. The observed-behaviour gap is suggestive rather than proof that the alternative would have worked.

## See Also

- [[markov-game]] · [[reinforcement-learning]] · [[value-iteration]]
- [[structured-model-decomposition]] · [[probability-surface]] · [[expected-possession-value]]
- [[counterfactual-simulation]] · [[eventgpt]] · [[scoutgpt]]
- [[expected-value-possession-framework|Source Summary]]
