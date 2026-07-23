---
title: "HPUS (Holistic Possession Utilization Score)"
type: concept
tags: [sports-analytics, action-valuation, player-evaluation, evaluation, point-process, event-stream-data]
sources: [raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 10%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# HPUS (Holistic Possession Utilization Score)

HPUS ([[transformer-point-process-football-event-modelling|Yeung, Sit & Fujii, 2023]]) evaluates how well a team uses a possession, using the [[nmstpp]] model's forecasts of expected zone, action, **and** time. It extends the poss-util metric from [[seq2event]], whose distinguishing limitation was ignoring timing.

## Holistic Action Score

Each action is scored:

$$\text{HAS} = \frac{\sqrt{E(\text{Zone} \mid H) \cdot E(\text{Action} \mid \text{Zone}, H)}}{t} \in [0, 10]$$

with the two expectations built from coarse ordinal scores:

$$E(\text{Zone} \mid H) = 0 \cdot P(\text{Area}_0) + 5 \cdot P(\text{Area}_1) + 10 \cdot P(\text{Area}_2)$$
$$E(\text{Action} \mid \cdot) = 0 \cdot P(\text{loss}) + 5 \cdot P(\text{dribble, pass}) + 10 \cdot P(\text{cross, shot})$$

Three design choices are doing real work here:

- **Multiplying zone by action** rather than adding them. A shot scores 10 on the action axis, but a shot from distance should not score like a shot from the six-yard box — multiplication lets position discount the action.
- **Dividing by inter-event time** rewards *efficiency*. The faster an action, the less time the opposition has to reorganise. This is the component only available because NMSTPP forecasts timing, and $t$ is floored at 1 to stop the score exploding.
- **Square root** keeps the result in $[0,10]$.

## Aggregating a Possession

$$\text{HPUS} = \sum_{i=1}^{n} \phi(n + 1 - i) \, \text{HAS}_i, \qquad \phi(x) = e^{-0.3(x-1)}$$

Weights decay exponentially *backwards from the last action*, so the possession's culmination dominates while earlier build-up still contributes. The decay constant 0.3 was chosen to give meaningful weight to 5–6 events, matching the observed average possession length of 5.2.

**HPUS+** counts only possessions that end in an attack (cross or shot), separating *opportunities created* from *opportunities converted*.

## Validation

Correlations across the 2017/18 Premier League:

| Metric | vs final ranking | vs goals | vs xG |
|---|---|---|---|
| Goals | −0.84 | — | 0.97 |
| [[expected-goals\|xG]] | −0.81 | 0.97 | — |
| **HPUS** | **−0.78** | **0.92** | **0.92** |
| HPUS+ | −0.77 | 0.91 | 0.90 |

HPUS tracks league position nearly as well as goals and xG — while **never using goal or shot-outcome data at any stage**, neither in the NMSTPP model nor in the metric. That is the substantive claim: possession quality is legible from event dynamics alone.

A curious regularity: every team's HPUS ratio (HPUS+ / HPUS) sits near **0.30**, suggesting a near-universal conversion rate from created opportunity to actual attack, largely independent of team quality. Teams differ in how many opportunities they generate, not in what fraction they convert.

## What It Adds Over Other Metrics

HPUS occupies a different niche from the [[action-valuation]] cluster:

| | Unit of analysis | Uses outcomes? | Models time? |
|---|---|---|---|
| [[expected-goals\|xG]] | Shot | Yes (goals) | No |
| [[expected-threat\|xT]] | Action → player | Yes (via xG term) | No |
| [[vaep]] | Action → player | Yes (goals) | Partially (features) |
| **HPUS** | **Possession → team** | **No** | **Yes** |

Because it is possession- and team-level, HPUS cannot do player valuation — the primary use of xT and VAEP. Conversely it can characterise match dynamics that action-level metrics miss: the paper shows Newcastle United creating opportunities against Manchester City but failing to convert them (high HPUS, low HPUS+), while against Chelsea they both created and converted more — despite both matches ending 3:1.

## Limitations

- The scoring weights (0/5/10), the time floor, and the 0.3 decay are hand-chosen rather than learned.
- Possession- and team-level only; no player attribution.
- Attacking analysis only — defensive contribution is out of scope, as only the possessing team's on-ball events are modelled.

## See Also

- [[nmstpp]]
- [[seq2event]]
- [[action-valuation]]
- [[expected-goals]]
- [[expected-threat]]
- [[vaep]]
- [[action-valuation-frameworks-compared]]
- [[transformer-point-process-football-event-modelling|Source Summary]]
