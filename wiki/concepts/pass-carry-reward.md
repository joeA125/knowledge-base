---
title: "Pass Carry Reward (PCR)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, event-stream-data, recruitment, transfer-prediction, evaluation, single-source]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 75%
  inferred: 20%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Pass Carry Reward (PCR)

PCR is [[andrei-shelopugin|Shelopugin's]] season-level player metric: the sum of a player's [[expected-possession-value|EPV]] changes across all passes and carries, normalised per 60 minutes of [[effective-playing-time|effective playing time]].

$$PCR(p) = \frac{60 \sum \Delta EPV(e_i \mid p,\; e_i \in \{\text{pass}, \text{carry}\})}{\sum \text{minutes}}$$

"Carry" is broad here — dribbles, ball carries, and simply holding possession without moving much.

## What Makes It Different

Structurally PCR is the familiar aggregation step: sum action values, divide by time. Three choices distinguish it from [[vaep]] or [[expected-threat|xT]] per-90 ratings.

**The denominator is effective time.** Nearly every other rating in the vault normalises on clock minutes. See [[effective-playing-time]] for why this is not cosmetic — it changes which players look productive.

**The action values are time-decayed.** $\Delta EPV$ inherits the [[temporal-discounting|discount factor]], so credit falls off continuously with distance from the shot rather than at a fixed action count.

**It is deliberately restricted to passes and carries.** Shots and set pieces are excluded from the sum even though the underlying model values them. The metric is aimed squarely at *ball progression* — the skill the author wants for the selection problem — not at total contribution.

## The Metric Is Never Validated

This is the honest reading of the source, and worth stating plainly because it is easy to mistake the paper's headline results for validation of PCR.

The paper reports strong RMSE improvements over a persistence baseline. Those results validate the **forecasting model** that predicts next-season PCR — they say the metric is *predictable*, not that it is *correct*. The author is explicit that no mathematical demonstration that EPV-based metrics track player skill is available, and proposes two unexecuted routes: expert assessment of shortlists, and comparison against real transfers to elite clubs on the assumption the top of the market is efficient.

Predictability without construct validity is a real gap. A metric measuring something stable but irrelevant would also forecast well. Compare [[predictive-validity]], where the test is forecasting an *independent* outcome rather than the metric's own future value.

## Reading the Reported Values

The published top-PCR seasons are instructive about what the metric rewards:

| Player | Season | PCR |
|---|---|---|
| Ziyech (Ajax) | 2019/20 | 0.355 |
| Iličić (Atalanta) | 2020/21 | 0.315 |
| Zoubir (Qarabağ) | 2020/21 | 0.314 |
| Doku (Man City) | 2023/24 | 0.299 |

Two patterns. First, the list is dominated by **wide creators and dribblers** — consistent with the pass/carry restriction and with the offensive bias noted across [[action-valuation]]. Second, players from weaker leagues (Azerbaijan, Norway, Malta, Northern Ireland) appear alongside elite-league players at similar raw values, which is precisely why the forecasting layer needs [[league-strength-rating|league strength features]]: raw PCR is **not** cross-league comparable.

## Relation to Other Season Metrics

| | PCR | [[hpus]] | poss-util | VAEP per 90 |
|---|---|---|---|---|
| Unit | Player | Team | Team | Player |
| Uses goal/shot data | Yes (via xG) | **No** | No | Yes |
| Time normalisation | **Effective** | Per match | Per match | Clock |
| Credit decay | **Exponential** | — | — | Fixed window |
| Includes duels | **Yes**, separately | No | No | No |

## Limitations

- **Offensive bias**, shared with the whole [[action-valuation]] family — defenders accrue little pass/carry reward.
- **Context dependence.** Raw PCR reflects league, team, and role as much as player. The paper's answer is to model it downstream rather than to remove it from the metric.
- **No reliability figure.** [[split-half-reliability]] is not reported, so PCR cannot be placed on the xT-versus-VAEP stability axis that matters most for [[recruitment]].
- **Single source**, unreviewed preprint.

## See Also

- [[expected-possession-value]] · [[action-valuation]]
- [[effective-playing-time]] · [[temporal-discounting]]
- [[transfer-performance-prediction]] · [[league-strength-rating]]
- [[predictive-validity]] · [[recruitment]]
- [[epv-control-duel-skills-football|Source Summary]]
