---
title: "Possession Risk"
type: concept
tags: [sports-analytics, action-valuation, discounting, markov-model, player-evaluation, event-stream-data, evaluation]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/evaluating-football-player-actions.md]
confidence: 0.8
provenance:
  extracted: 55%
  inferred: 40%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Possession Risk

The downside half of [[action-valuation]]: an action does not only change your team's chance of scoring, it changes the opponent's. A risky forward pass that is intercepted hands over the ball in a dangerous area; a safe backward pass does not.

Modelling this is what separates the frameworks in this vault more sharply than any other design choice.

## Risk-Reward as an Old Idea

The framing goes back to Pollard & Reep (1997), who proposed valuing a possession as the [[expected-goals|xG]] it generates minus the xG the opponent generates in the *following* possession. The structure — reward minus subsequent risk — has been inherited by everything since.

Where frameworks differ is the **horizon** over which risk is counted:

| Framework | Risk horizon | Consequence |
|---|---|---|
| [[expected-threat\|xT]] | None — possession-bounded | Cannot value risk at all |
| Pollard & Reep | Exactly the next possession | Crude but explicit |
| [[vaep]] | Next $k = 10$ actions, crossing turnovers | Risk-aware; the reason VAEP is "action-based" |
| Shelopugin | **Unbounded, [[temporal-discounting\|decayed]]** | Risk fades continuously with time |

xT's inability to model risk is not an oversight but a structural consequence of stopping at the possession boundary — conceding happens *after* the possession ends, so a strictly possession-bounded model has no way to see it.

## Two Problems With the Fixed Horizon

[[andrei-shelopugin|Shelopugin]] identifies two failures of the "exactly the next possession" convention.

**Elapsed time is ignored.** A player completes a pass; his team loses the ball ten seconds later; the opponent wins a penalty twenty seconds after that. Under the classical rule the passer is charged the full penalty xG, $-0.75$. This is plainly wrong — two intervening events and thirty seconds separate his action from the outcome.

Applying decay gives $-0.95^{30} \times 0.75 \approx -0.16$, which reflects genuine but attenuated responsibility.

**One possession is the wrong count.** A team can lose the ball, win it back, and concede on the *third* possession — or score on it. Counting exactly one subsequent possession makes the model blind to this, and the choice of "one" is arbitrary. Any fixed count is.

## The Resolution

Once value is [[temporal-discounting|decayed by elapsed time]], the possession count stops mattering. The target becomes the decayed sum of all future team xG minus the decayed sum of all future opponent xG, over an unbounded horizon:

$$PV(c_i) = \sum_{\text{team}}\left(1 - \prod_j (1 - \gamma^{(t_j - t_i)} xG_j)\right) - \sum_{\text{opponent}}\left(1 - \prod_j (1 - \gamma^{(t_j - t_i)} xG_j)\right)$$

Discounting does the work that the horizon cutoff was previously doing, but continuously rather than discretely. This mirrors the standard argument for discounting in infinite-horizon [[reinforcement-learning]].

## Risk Enters the Reward Twice

In the $\Delta EPV$ reward formulation, a turnover penalises the player **twice over**: once because the possession's positive value is lost, and again because the opponent's expected value now counts against him. The paper describes this as intentional.

Whether it is correct is arguable. Double-counting is defensible if you believe turnovers in dangerous areas are underpunished by conventional metrics — a common view — but it is a value judgement embedded in the target rather than a property estimated from data.

## The Structural Consequence

Risk modelling is the main reason the reliability ordering in this vault comes out as it does. [[vaep]], which models risk, replicates at $\rho = 0.25$; [[expected-threat|xT]], which does not, replicates at $\rho = 0.89$. Risk terms are estimated from rarer, noisier events — turnovers leading to opposition chances — so incorporating them buys realism and pays in stability.

Shelopugin's unbounded decayed horizon takes this further in the realism direction. No reliability figure is reported for [[pass-carry-reward|PCR]], so where it lands on that trade-off is unknown, and it is the single most useful missing number in the paper.

## See Also

- [[temporal-discounting]] · [[expected-possession-value]] · [[action-valuation]]
- [[vaep]] · [[expected-threat]] · [[expected-goals]]
- [[split-half-reliability]] · [[pass-carry-reward]]
- [[action-valuation-frameworks-compared]]
- [[epv-control-duel-skills-football|Source Summary]]
