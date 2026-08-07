---
title: "Data Stadium Inc."
type: entity
tags: [entity, organisation, data-provider, sports-analytics, event-stream-data, optical-tracking-data, off-ball, construct-validity, selection-bias]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md]
confidence: 0.75
provenance:
  extracted: 58%
  inferred: 33%
  generated: 5%
  imported: 0%
  ambiguous: 4%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Data Stadium Inc.

Japanese sports data provider, supplier of the event and tracking data behind **three held sources**, all from the [[nagoya-university|Nagoya]] group and all on the 2019 Meiji Yasuda J1 League. Events at 30 Hz, tracking of all players and the ball at 25 Hz.

| Source | Coverage |
|---|---|
| [[football-defence-evaluation-vdep\|VDEP]] | 45 matches |
| [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | 34 Yokohama F. Marinos matches |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | 54 matches, including all 34 Yokohama matches |

VDEP's data was distributed jointly with the Research Center for Medical and Health Data Science at the Institute of Statistical Mathematics via a research competition.

## Why It Is Worth Recording

**It is the vault's only non-Western data provider.** Everything else here runs on [[stats-perform|STATS LLC / Opta]], Wyscout, or StatsBomb, covering European and South American football. A J-League source is a meaningful widening — the vault's findings otherwise describe elite European football and generalise elsewhere only by assumption, which is the data-availability instance of [[selection-bias]] noted across these pages.

**The access route is unusual.** The VDEP data reached the researchers through an academic competition rather than a commercial licence, under a contract between the league and the provider rather than with players. Materially different from the club-embedded access behind [[expected-value-possession-framework|the Barcelona EPV work]] or the vendor licensing behind [[vaep]], and one of the few realistic routes to tracking data for academic researchers without a club affiliation.

**Provider conventions are load-bearing.** VDEP's definitions of *ball recovery* and *effective attack* are built on this provider's 19-type event taxonomy. As with [[duel-skill-rating|duel-winner definitions]] and the vendor differences that motivated [[spadl]], a metric defined over an annotation scheme inherits that scheme's judgements.

Nakahara et al. show the other side of the same coin: they take **pass and shot labels from this provider's event stream** as two of their 14 actions and **discard the dribbling and trapping labels** that also exist in it. So the provider's taxonomy sets the ceiling on the action vocabulary, and the researchers set the floor. See [[action-space-design]].

## The Provider That Made the Vault's Only Metric Comparison Possible

> **Added 2026-08-07.**

Two held metrics — [[c-obso|C-OBSO]] and Nakahara et al.'s Q-values — are computed on **the same 34 Yokohama F. Marinos matches from this provider**, which is what allows them to be correlated at all. The result, $\rho = 0.182$, is the vault's only head-to-head between two off-ball metrics. See [[construct-validity]] and [[off-ball-value]].

The general point is worth extracting: **shared data, not shared method, is what makes comparison possible.** The vault's long-standing complaint that nobody benchmarks across frameworks ([[action-valuation-frameworks-compared]]) is partly a complaint about data access — comparison requires two metrics on one dataset, and licensing usually prevents it. Where a single provider serves a single group repeatedly, comparison becomes available almost by accident.

That also bounds what this comparison shows. One club, one season, 14 players, one provider's conventions.

## See Also

- [[event-stream-data]] · [[optical-tracking-data]] · [[spadl]] · [[action-space-design]]
- [[stats-perform]] · [[selection-bias]] · [[vdep]] · [[c-obso]] · [[off-ball-value]] · [[construct-validity]]
- [[nagoya-university]] · [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[masakiyo-teranishi]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al. Summary]]
