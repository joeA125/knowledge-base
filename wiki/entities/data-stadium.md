---
title: "Data Stadium Inc."
type: entity
tags: [entity, organisation, data-provider, sports-analytics, event-stream-data, optical-tracking-data, off-ball, construct-validity, selection-bias, action-space, reinforcement-learning]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/action_valuation_football_agentic_reinforcement_learning.md, raw/papers/adaptive_action_supervision_multi_agent_reinforcement.md]
confidence: 0.8
provenance:
  extracted: 60%
  inferred: 32%
  generated: 5%
  imported: 0%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-08-07
---

# Data Stadium Inc.

Japanese sports data provider, supplier of the event and tracking data behind **four held sources**, all from the [[nagoya-university|Nagoya]] group and all on the 2019 Meiji Yasuda J1 League. Events at 30 Hz, tracking of all players and the ball at 25 Hz.

| Source | Coverage |
|---|---|
| [[football-defence-evaluation-vdep\|VDEP]] | 45 matches |
| [[creating-scoring-opportunities-trajectory-prediction\|C-OBSO]] | 34 Yokohama F. Marinos matches |
| [[action-valuation-multi-agent-reinforcement-learning\|Nakahara et al.]] | 54 matches, including all 34 Yokohama matches |
| [[adaptive-action-supervision-multi-agent-rl\|Fujii et al.]] | **The same 54 matches** — 198 last-pass-and-goal and 1,385 last-pass sequences |

VDEP's data was distributed jointly with the Research Center for Medical and Health Data Science at the Institute of Statistical Mathematics via a research competition.

## Why It Is Worth Recording

**It is the vault's only non-Western data provider.** Everything else here runs on [[stats-perform|STATS LLC / Opta]], Wyscout, or StatsBomb, covering European and South American football. A J-League source is a meaningful widening — the vault's findings otherwise describe elite European football and generalise elsewhere only by assumption, the data-availability instance of [[selection-bias]].

**The access route is unusual.** VDEP's data reached the researchers through an academic competition rather than a commercial licence, under a contract between the league and the provider rather than with players. Materially different from the club-embedded access behind [[expected-value-possession-framework|the Barcelona EPV work]] or the vendor licensing behind [[vaep]], and one of the few realistic routes to tracking data for academics without a club affiliation.

**Provider conventions are load-bearing.** VDEP's definitions of *ball recovery* and *effective attack* are built on this provider's 19-type event taxonomy. As with [[duel-skill-rating|duel-winner definitions]] and the vendor differences that motivated [[spadl]], a metric defined over an annotation scheme inherits that scheme's judgements.

## The Same Data, Four Different Extractions

> **Extended 2026-08-07.** The fourth source makes a pattern visible that three did not.

Every one of the four **subsets this data differently**, and each subset is a modelling choice nobody defends:

| Source | Unit extracted | Filter |
|---|---|---|
| VDEP | Events | All |
| C-OBSO | 412 sequences | **Shot-ending only** |
| Nakahara et al. | ~2,900 possessions | **Attacking third only**, 50–300 frames |
| Fujii et al. | 1,583 sequences | **Last-pass sequences only** |

C-OBSO takes shot-ending sequences; Fujii et al. take last-pass sequences whether or not a goal followed; Nakahara et al. take attacking-third possessions regardless of how they end. **Three overlapping but non-identical populations of football moments, from one dataset, in one research group.**

That is a partial explanation for the vault's most striking number — C-OBSO and Nakahara's Q-values correlating at $\rho = 0.182$. Metrics computed over different populations of moments need not agree, and the naming ("off-ball contribution") conceals the difference. See [[construct-validity]].

The action vocabulary diverges too. Nakahara et al. take pass and shot labels from the event stream and **discard the dribbling and trapping labels**; Fujii et al. split passing into **high and short** and drop sprint states. So the provider's taxonomy sets the ceiling and each paper sets its own floor. See [[action-space-design]].

## The Provider That Made the Vault's Only Metric Comparison Possible

Two held metrics — [[c-obso|C-OBSO]] and Nakahara et al.'s Q-values — are computed on **the same 34 Yokohama F. Marinos matches from this provider**, which is what allows them to be correlated at all. The result, $\rho = 0.182$, is the vault's only head-to-head between two off-ball metrics. See [[off-ball-value]].

The general point is worth extracting: **shared data, not shared method, is what makes comparison possible.** The complaint that nobody benchmarks across frameworks ([[action-valuation-frameworks-compared]]) is partly a complaint about data access — comparison requires two metrics on one dataset, and licensing usually prevents it. Where a single provider serves a single group repeatedly, comparison becomes available almost by accident.

**But availability is not sufficient.** Four papers now share this dataset and only one pair has been compared. The other five pairings are equally computable and equally unrun.

That also bounds what the one comparison shows: one club, one season, 14 players, one provider's conventions, and — per the table above — not even the same subset of moments.

## See Also

- [[event-stream-data]] · [[optical-tracking-data]] · [[spadl]] · [[action-space-design]] · [[selection-bias]]
- [[stats-perform]] · [[vdep]] · [[c-obso]] · [[off-ball-value]] · [[construct-validity]] · [[action-valuation-frameworks-compared]]
- [[nagoya-university]] · [[keisuke-fujii]] · [[hiroshi-nakahara]] · [[masakiyo-teranishi]] · [[kazushi-tsutsui]] · [[nfootball]]
- [[football-defence-evaluation-vdep|VDEP]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] · [[action-valuation-multi-agent-reinforcement-learning|Nakahara et al.]] · [[adaptive-action-supervision-multi-agent-rl|Fujii et al.]]
