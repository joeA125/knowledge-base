---
title: "Game Theory"
type: concept
tags: [game-theory, statistics, machine-learning, sports-analytics, policy-modelling, counterfactual, reinforcement-learning, evaluation]
sources: [raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Game Theory

The study of strategic interaction: what agents should do when their payoff depends on what *other* agents do. Formally, a set of players, a strategy set for each, and a payoff function mapping strategy profiles to outcomes.

The vault's football sources are almost entirely **single-agent** — they value one player's action, or one team's possession, treating opponents as environment. Game theory is the framework for treating them as agents.

## The Core Objects

| Object | Definition |
|---|---|
| **Strategy profile** $s = (s_i, s_{-i})$ | One strategy per agent; $s_{-i}$ denotes everyone but $i$ |
| **Payoff** $u_i(s)$ | Agent $i$'s value under that profile |
| **Best response** | $\arg\max_{s_i} u_i(s_i, s_{-i})$ |
| **Nash equilibrium** | A profile where every agent is playing a best response |

$$s^* \text{ is Nash} \iff \mathbb{E}[u_i(s_i^*, s_{-i}^*)] \ge \mathbb{E}[u_i(s_i', s_{-i}^*)] \;\; \forall s_i', \forall i$$

The defining property is **mutual** optimality: no agent gains by deviating *unilaterally*. That is weaker than collective optimality, and stronger than each agent optimising in isolation.

A **zero-sum** game has $u_1 = -u_2$ — one agent's gain is the other's loss. Football shot-taking is naturally zero-sum, which simplifies the analysis considerably.

## Why It Is Not Reinforcement Learning

The two are often conflated because both concern optimal action. The distinction matters:

| | [[reinforcement-learning\|Reinforcement learning]] | Game theory |
|---|---|---|
| Other agents | Folded into the environment | **Modelled as agents** |
| Solution concept | Optimal policy against a fixed world | **Equilibrium** against a reasoning opponent |
| Requires | Reward signal, many interactions | **Payoffs for each strategy profile** |
| Explains *why* | Poorly, without further analysis | By construction — the payoff table is the explanation |

[[optimal-decisions-shot-taking-situations|Yeung & Fujii]] choose game theory for the last row explicitly: RL "excels in learning policies and determining optimal decisions" but "often lacks the ability to explicitly explain why a specific decision is considered optimal without supplementary manual analysis."

The practical constraint is the third row. Game theory needs a payoff for **every** strategy combination, including ones never observed. That is why it has historically been confined in football to **penalty kicks** — a naturally isolated two-agent event with a small strategy space.

## The Move That Makes It Work Here

The obstacle to game theory in open play is that payoffs for unobserved strategy profiles are unknown. The solution is to **estimate payoffs with machine learning**:

$$\text{payoff}(\text{Shoot}, \text{Block}) = xSOT_{(B)}, \qquad \text{payoff}(\text{Shoot}, \text{Not Block}) = xSOT_{(NB)}$$

where the second is computed *counterfactually* — running the same model with the closest defender removed. See [[xsot]].

This is the general recipe, and it generalises past sport: **where payoffs are unobservable, learn them; then solve the game on the learned payoffs.** It inherits the learned model's errors, so the equilibrium is only as trustworthy as the payoff estimates.

## The Result, and Why It Is Unusual

Across 1,468 World Cup 2022 shot situations:

| | Defender: Block | Defender: Not Block |
|---|---|---|
| **Shoot** | 0.0866 | **0.2508** |
| **Pass** | **0.2456** | 0.2481 |

**Nash equilibrium: (Pass, Block).**

What makes this notable for the vault is not the specific answer but its *form*. Every other framework here describes what a player did and assigns it a value. This says what a player **should have done**, and can defend the claim by pointing at the payoff table.

The gap of 0.159 between passing and shooting against a blocking defender is read as evidence that **shooters shoot too much** — a claim about systematic decision error, which no other held source makes.

## Assumptions, and How Much They Bite

Three, all stated by the source:

**Rationality.** Agents choose best responses. Footballers make split-second decisions under fatigue; this is an idealisation, and the finding that shooters deviate from equilibrium is itself evidence against it.

**Complete information.** Every agent knows the whole game, and knows that everyone knows. Plainly false — players do not observe payoff tables. Whether it is *harmfully* false depends on whether trained intuition approximates the payoffs, which is untested.

**Static one-stage.** Both choose once, simultaneously. Justified here by a chi-square test showing consecutive shot outcomes are independent ($p = 0.96$), and by the absence of velocity data. But shot-taking is manifestly dynamic — the defender closes as the shooter decides.

The deeper limitation is **strategy-space coarsening**. "Pass" collapses ten possible recipients into one option; "Block" collapses every defensive posture. An equilibrium over a two-by-two game is a statement about a heavily simplified game.

## Elsewhere

Beyond sport, the same structure appears wherever outcomes depend on an opponent who is also optimising: auction design, pricing under competition, security and adversarial ML, and negotiation. The football case is a good illustration of the general difficulty — **the hard part is rarely solving the game, it is obtaining payoffs for strategy profiles nobody chose.**

Within sport, prior work has focused on penalty kicks (Palacios-Huerta, 2003; Tuyls et al., 2021), where the two-agent structure is native. Extending to open play is this paper's contribution.

## See Also

- [[xsot]] · [[optimal-decisions-shot-taking-situations|Source Summary]]
- [[policy-modelling]] · [[reinforcement-learning]] · [[markov-game]] · [[counterfactual-baseline]]
- [[theory-based-modelling]] · [[action-valuation]] · [[rare-event-proxy-targets]]
- [[calvin-yeung]] · [[keisuke-fujii]]
