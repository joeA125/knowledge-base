---
title: "Temporal Discounting"
type: concept
tags: [discounting, reinforcement-learning, temporal-difference, action-valuation, sports-analytics, dynamic-programming, statistics, time-series, model-selection]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/football-performance-time-series.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.8
provenance:
  extracted: 58%
  inferred: 34%
  generated: 6%
  imported: 0%
  ambiguous: 2%
lifecycle: draft
created: 2026-07-27
updated: 2026-08-07
---

# Temporal Discounting

Weighting a reward by how far away in time it is, usually geometrically: a reward $r$ arriving $\Delta t$ later counts as $\gamma^{\Delta t} r$ for some $\gamma \in (0,1)$. Introduced to economics by Samuelson (1937) and inherited by [[reinforcement-learning]], where it is standard in every value function.

## Three Different Reasons to Discount — and One Not To

The formula is the same in each case; the justification is not, and conflating them causes confusion.

**The economic/RL reason — impatience and convergence.** A reward now is worth more than the same reward later, and discounting also guarantees that infinite-horizon sums converge. Here $\gamma$ encodes a genuine preference for earlier payoff.

**The credit-assignment reason — attribution decay.** [[andrei-shelopugin|Shelopugin]] is explicit that his use is *not* the first one. He is not claiming a team should prefer to score sooner. The claim is evidential:

> if an action does not lead to a shot soon, it probably did not meaningfully advance the attack.

So $\gamma^{\Delta t}$ is a statement about **how much of the eventual goal this action deserves credit for**, not about when the goal should arrive.

**The declined case — bounded episodes with terminal reward.** [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] set $\gamma = 1$, citing Liu et al. (2020), "for simplicity". This is the one place in the vault where an RL framework *could* discount and chooses not to, and it is worth understanding why it is safe there and what it costs. See below.

The credit-assignment reading is the one relevant to [[action-valuation]], and it is why the discount is applied over *elapsed time* rather than over decision steps.

## Why It Replaces the Fixed Window

Before discounting, [[expected-possession-value|EPV]]-style targets treated every action in a possession identically. [[vaep]] instead uses a hard window of $k = 10$ actions, so the tenth-previous action is fully in scope and the eleventh fully out.

Both are crude. Geometric decay makes the boundary continuous:

| Approach | Boundary | Problem |
|---|---|---|
| Whole possession, undecayed | Turnover | Every action equally responsible |
| Fixed $k$-action window ([[vaep]]) | Action count | Discontinuous; ignores elapsed time |
| Capped time decay ([[football-performance-time-series\|Mendes-Neves]]) | 1 minute, floored at 5 actions | Still has a hard cap |
| **Geometric time decay** (Shelopugin) | None — decays to zero | Requires choosing $\gamma$ |
| **Undiscounted, terminal reward only** ([[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]]) | Episode cap, 300 frames | **Every action in a 30 s possession equally responsible** |

The last row is a return to the first, by a different route. Nakahara et al. bound the horizon *structurally* — an episode is a possession, capped at 300 frames — rather than by decay, so the undecayed sum converges trivially. But within that window, credit is spread **flat**.

## The $\gamma = 1$ Case, Assessed

> **Added 2026-08-07.**

It is defensible on its own terms and probably inert. Reward arrives only at the terminal frame $T$ of a bounded episode, so the return is a single term, and there is nothing for a discount factor to do that the episode cap does not already do.

Two things are nonetheless worth recording.

**The flat-credit consequence is real.** An action in second 1 and an action in second 29 of the same possession are, other things equal, credited identically — which is precisely the attribution problem Shelopugin's decay exists to solve. The framework does not encounter it as an error because it never aggregates *within* a possession by action; it aggregates by *player across frames*, taking a mean. The credit-assignment question is sidestepped rather than answered.

**The bootstrap does the work instead.** In a [[temporal-difference-learning|TD]] framework the value of an early action is not the discounted terminal reward directly — it is the value of the *next* state, which is itself bootstrapped forward. Attribution decay is implicit in how far the network propagates signal, not explicit in a parameter. That is arguably a better solution than a hand-set $\gamma$, and arguably an unexamined one, since nobody reports how far the [[gated-recurrent-unit|GRU]] actually propagates.

## Three Values, No Conversation

| Framework | $\gamma$ | Justification | Weight at 30 s |
|---|---|---|---|
| [[andrei-shelopugin\|Shelopugin]] | 0.95 /s | Stylistic preference | 21% |
| Shelopugin's proposed range | 0.9 – 0.99 | Attacking philosophy | 4% – 74% |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | **1** | "Simplicity", citing Liu et al. | **100%** |

Two football valuation frameworks use the same symbol at 0.95 and 1, one calling it a choice about attacking philosophy and the other a simplification, **and neither acknowledges the other's position exists.** Neither reports what varying it does. See [[free-parameters-load-bearing]].

## What It Unlocks: Risk Beyond One Possession

The decay's most useful consequence is on [[possession-risk|risk modelling]]. Undecayed risk models penalise a player for the opponent's chance in the *immediately following* possession — so a pass, a turnover ten seconds later, and a penalty twenty seconds after that assigns the passer $-0.75$.

With decay that becomes $-0.95^{30} \times 0.75 \approx -0.16$. And because the weight falls off smoothly rather than being cut off at a possession boundary, the sum can run over **arbitrarily many** subsequent possessions.

Discounting is what makes an unbounded horizon tractable. This is the same role it plays in infinite-horizon RL, arrived at from a different direction.

**Nakahara et al. reach the cross-possession case differently and less generally**: a $-1$ conceding term fires if the opponent scores *immediately after* the possession ends. One possession of lookahead, undecayed, hard-bounded — the crude version Shelopugin's decay was built to replace.

## Choosing $\gamma$

Shelopugin uses $\gamma = 0.95$ per second of [[effective-playing-time|effective time]] and offers it as a **stylistic hyperparameter**: 0.9 for analysts who value vertical, direct attacking; 0.99 for patient build-up.

That framing is appealing but should be treated sceptically. It converts an unvalidated modelling choice into a feature, and no sensitivity analysis is reported. Given that $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$, the range spans nearly two orders of magnitude in the weight given to a thirty-second-old action. The rankings almost certainly are sensitive.

A defensible alternative would be to fit $\gamma$ by maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity|predictive validity]], rather than asserting it.

## Interaction with Effective Time

The decay is applied over [[effective-playing-time|effective playing time]], and this is load-bearing rather than incidental. A player who wins a penalty is credited heavily because the spot kick follows almost immediately in *ball-in-play* terms — even though several minutes of VAR review may separate them on the clock. Discounting over wall-clock time would erase the reward.

Nakahara et al. avoid the issue by construction: possessions are cut at ball recovery and ball loss, so dead time is outside every episode. **Structural episode definition and effective-time discounting are two solutions to one problem**, and the first is cleaner where the data supports it.

## See Also

- [[possession-risk]] · [[expected-possession-value]] · [[action-valuation]] · [[free-parameters-load-bearing]]
- [[effective-playing-time]] · [[pass-carry-reward]] · [[model-selection]]
- [[value-iteration]] · [[reinforcement-learning]] · [[temporal-difference-learning]] · [[markov-game]] · [[multi-agent-reinforcement-learning]]
- [[vaep]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
- [[epv-control-duel-skills-football|Shelopugin Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
