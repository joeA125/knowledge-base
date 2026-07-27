---
title: "Intent vs Outcome Valuation"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, feature-engineering, interpretability, recruitment]
sources: [raw/papers/football-performance-time-series.md, raw/papers/evaluating-football-player-actions.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md]
confidence: 0.8
provenance:
  extracted: 65%
  inferred: 30%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Intent vs Outcome Valuation

When an [[action-valuation]] model assigns a value to an action, it is implicitly answering an ambiguous question. Was that a *good decision*, or was it *well executed*? These are different properties of the same action, and most valuation frameworks silently conflate them.

A hopeful long ball that finds a teammate and a hopeful long ball that goes out of play were the same decision. A tap-in and a thirty-yard curler into the top corner were very different executions of a similarly-valued opportunity.

## The Mechanism

[[football-performance-time-series|Mendes-Neves, Meireles & Mendes-Moreira]] give a clean recipe: **partition the feature set by whether a feature encodes what happened after the player committed to the action.**

| Model | Outcome features | Answers |
|---|---|---|
| **I-VAEP** (Intent) | Withheld | Was this the right thing to attempt from this position? |
| **O-VAEP** (Outcome-aware) | Included | How much value did the player actually produce? |

In their implementation the switch is five features: `outcome` (did the action succeed), `endAngleToGoal`, `endDistanceToGoal`, `distanceToPost`, `distanceToBar`. The last two describe *where in the goal mouth a shot was placed* — pure execution quality, unknowable at the moment of the decision.

Train the identical model twice, once with and once without. The difference between the two ratings is a measure of execution.

## Why This Is a General Recipe

The idea does not depend on [[vaep|VAEP]], on [[random-forest|Random Forests]], or on this particular label. It requires only that the feature set can be partitioned by an **information-availability criterion**: would the player have known this at the moment of choosing?

This makes it applicable to any framework in the [[action-valuation]] family, and it is unusually cheap — the same architecture, the same data, one ablated feature group.

## What Each Model Is Good For

**Intent models suit [[recruitment|recruitment]] and development.** Decision-making is more stable and more transferable across contexts than finishing. A player whose intent value is high but outcome value is low is making good choices and executing poorly — a coachable profile, and potentially underpriced. The reverse pattern may signal a player riding a hot streak of conversion that will regress.

**Outcome models suit accountability and post-hoc analysis.** What actually happened is what actually happened. For assessing what a player *contributed* to results already achieved, execution should count.

The gap between them is itself diagnostic. In the paper's worked example on the Barcelona 3–0 Real Madrid sequence (December 2017), the intent/outcome gap is negligible for build-up passes and largest for the shot (0.18 against 0.406) — precisely because finishing quality is exactly what the intent model is blind to.

## Relation to the Reliability Problem

This connects to the vault's central measurement problem. [[split-half-reliability|Van Roy et al.]] found VAEP ratings replicate poorly across random halves of a season ($\rho = 0.25$), and diagnosed the cause as **goals being rare, high-value, and noisy**.

An intent model should be structurally less exposed to that failure. Removing outcome features removes the strongest channel through which rare conversion events dominate a rating, leaving behind the more stable component — where players choose to act from and what they choose to attempt, which [[split-half-reliability|is known to be highly consistent]].

> **Untested inference.** No source in this vault reports split-half reliability for I-VAEP. The prediction that intent ratings are more reliable than outcome ratings is a plausible consequence of two separate findings, not a measured result. It is a well-posed experiment that nobody in the vault's sources has run.

## Where Other Frameworks Sit

| Framework | Position |
|---|---|
| [[expected-threat\|xT]] | Effectively pure intent — values only the ball's zone transition, gives no credit for finishing at all |
| [[vaep]] | Conflated — outcome features are present but not isolated |
| [[expected-goals\|xG]] | Pure intent *by construction* — measures chance quality, deliberately excluding how the shot was struck |
| [[on-ball-value\|Residual OBV]] | Outcome-oriented, but residualised against expectation |

Seen this way, the long-standing xG debate — "is he a good finisher or is he getting good chances?" — is exactly the intent/outcome distinction applied to shots alone. I-VAEP/O-VAEP generalises the same question to all 21 [[spadl]] action types.

## Limitations

- **The partition is a judgement call.** `endDistanceToGoal` describes where a pass ended, which is partly intent (where it was aimed) and partly execution (whether it got there). Assigning it wholly to "outcome" is a defensible simplification, not a clean separation.
- **Intent is inferred, never observed.** The intent model does not know what the player meant to do; it knows what a player in that position typically achieves. A mishit cross that becomes a dangerous through-ball is scored on its pre-outcome features, which may not reflect the actual intention.
- **Off-ball intent is invisible**, as with all [[event-stream-data|event-based]] valuation.

## See Also

- [[action-valuation]]
- [[vaep]]
- [[expected-threat]]
- [[expected-goals]]
- [[on-ball-value]]
- [[split-half-reliability]]
- [[feature-engineering]]
- [[player-rating-time-series]]
- [[action-valuation-frameworks-compared]]
- [[football-performance-time-series|Source Summary]]
