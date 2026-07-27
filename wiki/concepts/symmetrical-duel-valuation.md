---
title: "Symmetrical Duel Valuation"
type: concept
tags: [sports-analytics, duel-analysis, action-valuation, player-evaluation, event-stream-data, single-source]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 70%
  inferred: 25%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Symmetrical Duel Valuation

A **symmetrical duel** is a contested event — aerial or ground — in which two opposing players compete for a ball that neither yet possesses. [[andrei-shelopugin|Shelopugin]] introduces the term to isolate the events that break an assumption every other [[action-valuation]] framework in this vault quietly relies on.

A dribble is explicitly *not* symmetrical: the dribbler has the ball.

## Why Duels Break Standard Valuation

The unifying equation $V(a_i) = Q(S_i) - Q(S_{i-1})$ requires knowing *whose* prospects $Q$ describes. For a pass or a carry this is trivial. For a 50/50 header it is undefined — during the duel, neither team is in possession, and the event belongs to two players on opposing teams simultaneously.

Frameworks handle this by ignoring it. [[expected-threat|xT]] values only ball progression between zones. [[vaep]] operates over [[spadl]], which represents duels as separate per-player actions rather than as one contested event. The consequence is that a substantial category of football — and the one where physical mismatch matters most — is either invisible or misattributed.

## The Two Mechanisms

### 1. Inheriting value from what follows

A duel takes the possession value of the first control action after it, signed by outcome:

$$PV(d_i) = \begin{cases} PV(e_{i+1}) & s_i = s_{i+1} \\ -PV(e_{i+1}) & s_i \neq s_{i+1} \end{cases}$$

Consecutive duels — a header flicked into another header — recursively inherit the same value. This sidesteps the attribution problem rather than solving it: the duel is valued by *what it led to*, which is the same lookahead logic [[expected-possession-value|EPV]] uses everywhere.

### 2. Conditioning on who is contesting it

The subtler contribution. Consider a pass into an aerial duel. Under a duel-blind model this is a mildly *negative* action — the passer has surrendered certainty for a coin flip.

But it is only a coin flip if the players are evenly matched. Feeding the **probability of winning the duel** (from [[duel-skill-rating]]) into the EPV model as a feature means a long ball to a dominant target head is correctly rewarded, and the same ball to a weak one correctly punished.

This is a partial solution to a problem the paper names directly: **splitting credit between the passer and the receiver.** For passes into duels, the split now reflects the receiver's actual ability. For *accurate* passes, the author concedes no event-data solution exists.

## The Circularity, and the Fix

There is a trap here. If a strong duellist's expected value is inflated by his own skill rating, then $\Delta EPV = \text{outcome} - \text{expectation}$ punishes him for being good — he must clear a higher bar to earn the same reward.

The paper avoids this by maintaining **two duel models**:

| Model | Includes player skill? | Used for |
|---|---|---|
| $EPV^{avg}_{duel}$ | No — average player | The **duellist's own** reward baseline |
| $EPV^{ind}_{duel}$ | Yes | The **passer's** reward for creating the duel |

So the duellist is compared against what an average player would have managed in that situation, while the passer is credited for who he actually picked out. The asymmetry is deliberate and is the mechanism that makes credit-splitting work.

## Evidence It Matters

The [[epv-control-duel-skills-football|Donnarumma case study]] is the demonstration. His long passes under pressure went mostly to Ibrahimović or Leão, into comparably difficult aerial situations. Skill-adjusted win probabilities: 61.1% for Ibrahimović, 35.8% for Leão.

A duel-blind model values the two passes at 0.00075 and 0.00077 — indistinguishable. The skill-aware model gives 0.00135 and 0.00069, roughly a factor of two. The tactical instruction was identical; the quality of the option was not.

## What It Does Not Cover

- **Defensive duels are still valued through the possession lens.** Winning a header is rewarded by what your team then does with it, so a defensive clearance into nothing scores near zero.
- **Only two-player contests.** Crowded aerial situations are not modelled as such, though the number of opponents does enter the context model.
- **The winner definition is a convention**, not a measurement: fouled player wins, else first touch, else whoever's team gains possession. Reasonable, but it makes ratings partly an artefact of the data provider's annotation rules.

## See Also

- [[duel-skill-rating]] · [[expected-possession-value]] · [[action-valuation]]
- [[possession-risk]] · [[pass-carry-reward]]
- [[spadl]] · [[vaep]] · [[expected-threat]]
- [[epv-control-duel-skills-football|Source Summary]]
