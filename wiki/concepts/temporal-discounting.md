---
title: "Temporal Discounting"
type: concept
tags: [discounting, reinforcement-learning, action-valuation, sports-analytics, dynamic-programming, statistics, time-series]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/football-performance-time-series.md]
confidence: 0.8
provenance:
  extracted: 60%
  inferred: 35%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Temporal Discounting

Weighting a reward by how far away in time it is, usually geometrically: a reward $r$ arriving $\Delta t$ later counts as $\gamma^{\Delta t} r$ for some $\gamma \in (0,1)$. Introduced to economics by Samuelson (1937) and inherited by [[reinforcement-learning]], where it is standard in every value function.

## Two Different Reasons to Discount

The formula is the same in both cases; the justification is not, and conflating them causes confusion.

**The economic/RL reason — impatience and convergence.** A reward now is worth more than the same reward later, and discounting also guarantees that infinite-horizon sums converge. Here $\gamma$ encodes a genuine preference for earlier payoff.

**The credit-assignment reason — attribution decay.** [[andrei-shelopugin|Shelopugin]] is explicit that his use is *not* the first one. He is not claiming a team should prefer to score sooner. The claim is evidential:

> if an action does not lead to a shot soon, it probably did not meaningfully advance the attack.

So $\gamma^{\Delta t}$ is a statement about **how much of the eventual goal this action deserves credit for**, not about when the goal should arrive. The same mathematics serves a different epistemic purpose.

This second reading is the one relevant to [[action-valuation]], and it is why the discount is applied over *elapsed time* rather than over decision steps.

## Why It Replaces the Fixed Window

Before discounting, [[expected-possession-value|EPV]]-style targets treated every action in a possession identically: a pass in the defensive third and the cutback that created the chance both received the full possession value. [[vaep]] instead uses a hard window of $k = 10$ actions, so the tenth-previous action is fully in scope and the eleventh fully out.

Both are crude. Geometric decay makes the boundary continuous:

| Approach | Boundary | Problem |
|---|---|---|
| Whole possession, undecayed | Turnover | Every action equally responsible |
| Fixed $k$-action window ([[vaep]]) | Action count | Discontinuous; ignores elapsed time |
| Capped time decay ([[football-performance-time-series\|Mendes-Neves]]) | 1 minute, floored at 5 actions | Still has a hard cap |
| **Geometric time decay** (Shelopugin) | None — decays to zero | Requires choosing $\gamma$ |

Action count is a poor proxy for time. Ten quick one-touch passes and ten slow possession-recycling passes span very different intervals, and only the time-based version distinguishes them.

## What It Unlocks: Risk Beyond One Possession

The decay's most useful consequence is on [[possession-risk|risk modelling]]. Undecayed risk models penalise a player for the opponent's chance in the *immediately following* possession — so a pass, a turnover ten seconds later, and a penalty twenty seconds after that assigns the passer $-0.75$.

With decay that becomes $-0.95^{30} \times 0.75 \approx -0.16$. And because the weight falls off smoothly rather than being cut off at a possession boundary, the sum can run over **arbitrarily many** subsequent possessions — capturing the case where a team loses the ball, wins it back, and scores two possessions later.

Discounting is what makes an unbounded horizon tractable. This is the same role it plays in infinite-horizon RL, arrived at from a different direction.

## Choosing $\gamma$

Shelopugin uses $\gamma = 0.95$ per second of [[effective-playing-time|effective time]] and offers it as a **stylistic hyperparameter**: 0.9 for analysts who value vertical, direct attacking; 0.99 for those who value patient *tiki-taka* build-up.

That framing is appealing but should be treated sceptically. It converts an unvalidated modelling choice into a feature, and no sensitivity analysis is reported — so it is unknown how much the resulting player rankings move as $\gamma$ varies. Given that $0.9^{30} = 0.04$ against $0.99^{30} = 0.74$, the range spans nearly two orders of magnitude in the weight given to a thirty-second-old action. The rankings almost certainly are sensitive.

A defensible alternative would be to fit $\gamma$ by maximising the resulting metric's [[split-half-reliability|reliability]] or [[predictive-validity|predictive validity]], rather than asserting it.

## Interaction with Effective Time

The decay is applied over [[effective-playing-time|effective playing time]], and the paper notes this is load-bearing rather than incidental. A player who wins a penalty is credited heavily because the spot kick follows almost immediately in *ball-in-play* terms — even though several minutes of VAR review and protest may separate them on the clock. Discounting over wall-clock time would erase the reward.

## See Also

- [[possession-risk]] · [[expected-possession-value]] · [[action-valuation]]
- [[effective-playing-time]] · [[pass-carry-reward]]
- [[value-iteration]] · [[reinforcement-learning]] · [[markov-game]]
- [[vaep]] · [[football-performance-time-series|Valuing Players Over Time Summary]]
- [[epv-control-duel-skills-football|Source Summary]]
