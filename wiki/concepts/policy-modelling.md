---
title: "Policy Modelling"
type: concept
tags: [policy-modelling, imitation-learning, markov-model, game-theory, sports-analytics, action-valuation, machine-learning, evaluation, counterfactual, reinforcement-learning, temporal-difference, action-space]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/optimal_football_decisions_shot_taking_situations.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 48%
  inferred: 44%
  generated: 6%
  imported: 0%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Policy Modelling

Estimating the policy agents **actually follow**, rather than solving for the one they should. In an MDP framing, learning $\pi(s,a) = \mathbb{P}(A = a \mid s)$ from observed behaviour and treating it as a quantity of interest.

## The Inversion

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] frame possession value as a Markov decision process and then state the departure plainly: the aim is not to find the optimal policy $\pi$, but to estimate value **under the average policy learned from historical data**.

This inverts the usual objective. Reinforcement learning treats the behaviour policy as something to improve, or to correct for via importance weighting. Here it is the estimand. It is the same move [[imitation-learning]] makes, reached from the valuation side rather than the control side.

Two reasons, and both generalise:

**The policy is itself the finding.** Knowing that a player in a given situation passes 29.3% of the time, drives 23.0%, and shoots 47.6% is *directly* useful to a coach. Where behaviour diverges from what the value model rewards, that gap is the coaching intervention.

**The optimal-policy counterfactual is usually unfounded.** An optimal-policy value function answers "what is this position worth to a perfect player?" No data contains perfect play, so there is nothing to learn one from.

## The Choice Reappears as an Algorithm Choice

> **Added 2026-08-07.** [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] make the same commitment as Fernández et al., but structurally rather than by declaration — which is worth recording because it shows the choice is not merely a framing preference.

Their [[temporal-difference-learning|TD]] algorithm is **SARSA**, which is *on-policy*: the bootstrap target is $Q(s_{t+1}, a_{t+1})$, using the action the player actually took next, rather than $\max_a Q(s_{t+1}, a)$.

| | SARSA (on-policy) | Q-learning (off-policy) |
|---|---|---|
| Target uses | The action actually taken | The **best** action |
| Converges toward | Value of the **behaviour policy** | Value of the **optimal policy** |
| As a metric | "What was this worth, given how they play?" | "What would this be worth, played perfectly?" |

So the vault now holds two frameworks committed to average-policy value by entirely different mechanisms — one by choosing an estimand, one by choosing an update rule. **A one-line algorithmic substitution would convert the second into an optimal-policy framework**, and the paper does not discuss the choice.

There is a tension it leaves unresolved. The framework's headline capability is reporting Q-values for actions **nobody took**, yet SARSA never targets those values; they are shaped by [[action-supervision]] and network smoothness instead. The counterfactual arm is on-policy in its training signal and off-policy in its use.

## When the Counterfactual *Is* Founded

**Correction, 2026-07-27.** An earlier revision of this page — and of [[reinforcement-learning]] and [[value-iteration]] — stated that this literature does not solve for optimal policy at all. That is now too strong.

[[optimal-decisions-shot-taking-situations|Yeung & Fujii (2024)]] compute one, and the mechanism shows exactly when the objection above does and does not bite. They restrict to a **two-action game** — the shooter chooses {Shoot, Pass}, the defender {Block, Not Block} — and estimate a payoff for each of the four profiles, including the two never observed, by re-running the models with the closest defender removed.

The counterfactual is founded here because the strategy space is small enough to *enumerate* and each cell is estimable from data. The objection stands wherever the action space is large or continuous.

So the correct statement is narrower and more useful: **optimal-policy analysis is available exactly where the strategy space can be coarsened enough to enumerate, and coarsening is the price.**

**Sharpened 2026-08-07.** Nakahara et al. sit between the two — 14 actions per agent, enumerable but not tiny. Their counterfactuals are founded in the weak sense that every cell has *a* value, and unfounded in the strong sense that most cells were never visited. **Enumerability is necessary and not sufficient; what matters is whether each enumerated cell is estimable.** See [[action-space-design]].

## Where It Enters the Decomposition

Two policy components appear in the [[structured-model-decomposition|EPV decomposition]]:

| Component | Question | Output |
|---|---|---|
| Action selection $\mathbb{P}(A\|T_t)$ | Pass, drive, or shoot? | 3-way softmax |
| Pass selection $\mathbb{P}(D_t\|A=\rho, T_t)$ | Pass *where*? | [[probability-surface\|Surface summing to 1]] |

Both act as **weights** on the corresponding value estimates. EPV is the value of each option weighted by how likely it is to be taken — an expectation over the behaviour policy, not a maximum over options.

Pass selection is a subtle and easily-missed distinction from pass *success*. Success asks "would this pass work?"; selection asks "would this pass be attempted?" A pass into a dangerous area might have high success probability and near-zero selection probability, because players do not think to play it. That gap is where the most interesting coaching output lives.

**A third route to the same object.** Nakahara et al. never estimate a policy explicitly, but $\text{softmax}(\mathbf{Q}_{s_t})$ is treated as one inside the [[action-supervision]] loss — an implicit behaviour policy read off a value function. That makes the policy and the value function the same parameters, which is convenient and means the two cannot be separately inspected.

## Value Under Policy vs Value Under Optimum

Fernández et al. report both, and the difference is the deliverable: policy-weighted EPV 0.032 against best-available 0.112.

The first says what this possession is worth given how footballers behave; the second what it is worth to someone who spots the best option. **The gap between them is the value being left on the pitch by ordinary decision-making.**

Yeung & Fujii reach a comparable conclusion by a different route. Against a blocking defender, passing is worth 0.2456 and shooting 0.0866 — a gap of 0.159, read as evidence that **shooters shoot too much.**

Two independent frameworks, one surface-based and one game-theoretic, both finding systematic divergence between observed and optimal behaviour. That is the closest thing this literature has to a prescriptive finding.

⚠️ **A third framework shows the gap can be tuned.** [[action-supervision|Nakahara et al.'s]] $\lambda_1$ weights an imitation loss against the value loss; the authors state that too much supervision makes the model overfit to observed actions and stop distinguishing counterfactuals. In that family, how suboptimal players look is partly a hyperparameter.^[generated: `optimality-gap-is-tunable`, declared on [[action-supervision]]. rests-on: source:nakahara-lambda-tradeoff] See [[observed-versus-optimal-decisions]].

Structurally all of this is the advantage function $A(s,a) = Q(s,a) - V(s)$, repurposed as an interpretive device rather than a training signal. It is also a [[counterfactual-baseline]] whose reference is the *optimum* rather than a predicted behaviour.

## Caveats

- **The policy is a population average.** It describes how players behave, not how *this* player behaves. Conditioning on identity would give individual policies — which is what [[eventgpt]] and [[scoutgpt]] do with player-conditioned generation. Nakahara et al. offer a middle position: ten agents, one per attacking slot, so policies are role-conditional but not identity-conditional.
- **Behaviour encodes coaching, not just judgement.** A player may pass backwards because instructed to. The policy conflates individual decision-making with team instruction, and nothing separates them.
- **Circularity risk.** If the value model is trained on outcomes generated under this policy, then "the policy is suboptimal by the value model's lights" is a claim about internal consistency, not about football. This is sharpest in the RL case, where an imitation prior on the policy is an *input* to the value function whose disagreement with the policy is then the finding.

## See Also

- [[game-theory]] · [[imitation-learning]] · [[counterfactual-baseline]] · [[reinforcement-learning]] · [[action-supervision]]
- [[markov-game]] · [[value-iteration]] · [[temporal-difference-learning]] · [[multi-agent-reinforcement-learning]] · [[action-space-design]]
- [[structured-model-decomposition]] · [[probability-surface]] · [[counterfactual-simulation]] · [[free-parameters-load-bearing]]
- [[expected-possession-value]] · [[xsot]] · [[eventgpt]] · [[scoutgpt]] · [[generative-model]] · [[observed-versus-optimal-decisions]]
- [[expected-value-possession-framework|Fernández Summary]] · [[optimal-decisions-shot-taking-situations|Yeung & Fujii Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
