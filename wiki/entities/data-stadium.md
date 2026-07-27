---
title: "Data Stadium Inc."
type: entity
tags: [entity, organisation, data-provider, sports-analytics, event-stream-data, optical-tracking-data, single-source]
sources: [raw/papers/football_defence_evaluation.md]
confidence: 0.65
provenance:
  extracted: 55%
  inferred: 35%
  ambiguous: 10%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Data Stadium Inc.

Japanese sports data provider, supplier of the event and tracking data behind [[football-defence-evaluation-vdep|the VDEP paper]] — 45 matches from the 2019 Meiji Yasuda J1 League, with events at 30 Hz and tracking of all players and the ball at 25 Hz.

Distributed jointly with the Research Center for Medical and Health Data Science at the Institute of Statistical Mathematics, an academic body, via a research competition.

## Why It Is Worth Recording

**It is the vault's only non-Western data provider.** Everything else here runs on [[stats-perform|STATS LLC / Opta]], Wyscout, or StatsBomb, covering European and South American football. A J-League source is a meaningful widening — the vault's findings otherwise describe elite European football and generalise elsewhere only by assumption, which is the data-availability instance of [[selection-bias]] noted across these pages.

**The access route is unusual and worth noting.** The data reached the researchers through an academic competition rather than a commercial licence, under a contract between the league and the provider rather than with players. This is a materially different arrangement from the club-embedded access behind [[expected-value-possession-framework|the Barcelona EPV work]] or the vendor licensing behind [[vaep]], and it is one of the few realistic routes to tracking data for academic researchers without a club affiliation.

**Provider conventions are load-bearing.** VDEP's definitions of *ball recovery* and *effective attack* are built on this provider's 19-type event taxonomy. As with [[duel-skill-rating|duel-winner definitions]] and the vendor differences that motivated [[spadl]], a metric defined over an annotation scheme inherits that scheme's judgements — so VDEP is not straightforwardly portable to a different provider's event stream without redefinition.

**Note:** vault knowledge of this organisation comes from the data-availability statement of a single paper.

## See Also

- [[event-stream-data]] · [[optical-tracking-data]] · [[spadl]]
- [[stats-perform]] · [[selection-bias]] · [[vdep]]
- [[football-defence-evaluation-vdep|Source Summary]]
