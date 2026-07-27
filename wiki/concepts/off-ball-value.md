---
title: "Off-Ball Value"
type: concept
tags: [off-ball, sports-analytics, action-valuation, defensive-valuation, player-evaluation, optical-tracking-data, probability-surface, pitch-control, counterfactual, evaluation]
sources: [raw/papers/expected_value_possession_framework.md, raw/papers/football_defence_evaluation.md]
confidence: 0.8
provenance:
  extracted: 50%
  inferred: 40%
  ambiguous: 10%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Off-Ball Value

Quantifying the contribution of players who do not have the ball — the runs that stretch a defence, the positioning that opens a passing lane, the shape a defence holds to deny space.

Long the **largest acknowledged gap** in this vault's football coverage; [[action-valuation-frameworks-compared|every modelling task]] lists "on-ball only" among its shared limitations. Two held sources now address it from opposite sides of the ball, and a third line — cited but not held — goes further than either.

## Why Event Data Cannot See It

An [[event-stream-data|event stream]] records actions. Off-ball contribution is, by definition, the absence of an action — a player who makes a decisive run and never receives the ball generates **no event at all**.

No modelling sophistication recovers this from event data. It is a data limitation, fixable only with [[optical-tracking-data|tracking]].

## Two Routes In the Held Sources

### Attacking: read a value surface at player positions

[[expected-value-possession-framework|Fernández, Bornn & Cervone]] obtain off-ball value almost for free, as a by-product of the [[probability-surface|pass EPV surface]].

If the surface gives the EPV resulting from a pass to *every* location, the value of a player **standing** at a location is immediately available. Aggregating positive potential EPV added maps where a team generates off-ball advantage.

No model of off-ball behaviour is required. The framework never predicts runs or assigns credit for movement — it computes what every location is worth and reads off who is standing somewhere valuable.

### Defending: put all 22 positions in the state

[[football-defence-evaluation-vdep|Toda et al.]] take the other route. [[vdep]]'s state $s_i = [a_i, o_i]$ appends all 22 players' coordinates plus each player's distance from the ball, **sorted by proximity**.

The sort is the enabling trick — it makes the representation permutation-invariant, so "the nearest defender" occupies a fixed feature slot regardless of identity, which is what lets a tree ensemble use it.

[[shap]] confirms these features carry the signal: the top contributor to $P_{recoveries}$ is the **nearest defender's distance from the ball**; to $P_{attacked}$, the **nearest attacker's $x$-position**.

### The two compared

| | Attacking (surface) | Defending (state) |
|---|---|---|
| Mechanism | Read value surface at a location | Include positions as model input |
| Output unit | **Player** | **Team only** |
| Requires a spatial model | Yes — [[soccermap]] | No |
| Cost | Substantial | Modest |
| What it measures | Positional value if the ball arrives | Contribution to recovery / preventing penetration |

The asymmetry in output unit is structural, not an oversight. A surface is read *at a player's position*, so it individuates naturally. Putting 22 positions into one classifier produces one number for the configuration, with no principled way to split it — which is why VDEP is team-level and its authors say so.

## The Third Route: Counterfactual Positioning

> **Provenance warning.** This section is built from **citation lists and an author's own overview article**, not from primary sources. None of these papers is held in `raw/`. Capability claims are the authors' own and are unverified here.

**Correction, 2026-07-27.** This page previously described combining the spatial and defensive routes as "the clearest open opportunity in this vault's football coverage" and stated that nobody had attempted it. That was wrong, and was inferred from VDEP's limitations section without checking the follow-up literature.

**Umemoto & Fujii (2023)**, *Evaluation of team defense positioning by computing counterfactuals using StatsBomb 360 data* (StatsBomb Conference), does approximately this. As described by Fujii: identify the location with the highest **OBSO** (off-ball scoring opportunity — Spearman, 2018), select the closest defender and his grid cell, then search which cell he could have occupied to reduce OBSO most. The output is the positioning that "reduced the threat the most".

Three things make it the missing piece:

- **It individuates.** The intervention is on one named defender's location, so the credit is his.
- **It is a value surface, not a classifier.** The whole Fujii off-ball line builds on OBSO, which puts it closer to [[probability-surface|Fernández et al.'s]] machinery than to VDEP's event classification.
- **It answers "what should he have done?"** rather than "how good was the defence?" — the same realised-versus-available shift that [[policy-modelling|pass surfaces]] make on the attacking side.

A companion line attacks the attacking-side omission below. **Teranishi, Tsutsui, Takeda & Fujii (2022/23)** evaluate movement by the difference between a player's actual trajectory and a predicted one, explicitly crediting **movement sacrificed for a teammate** — space creation. Fujii reports that this required enormous computation to evaluate all 22 players, since it needs a separate trajectory prediction per player.

## What the Held Sources Still Do Not Capture

Narrowed by the above, but real for the frameworks actually read here:

**Credit for creating the space.** If a striker's run drags a centre-back away and a midfielder exploits the gap, the surface rewards the midfielder's location. The Teranishi trajectory work targets exactly this; the held sources do not.

**Movement over time.** Value is read frame by frame. Arriving in a good position through a clever run and standing there passively score identically.

**Individual defensive credit.** VDEP measures the defence collectively. Addressed by Umemoto & Fujii, unverified here.

**Errors of omission.** A defender who fails to press generates no event. Counterfactual positioning may cover this in principle — a better available cell implies the actual one was worse — but whether it does in practice is unverified.

So for the sources this vault has actually read: **positional value on the attacking side, collective value on the defending side.** The literature has moved further than that; the vault has not.

## Applications in the Held Sources

**Pressing analysis.** Off-ball advantage heatmaps for the formations most used against Liverpool's buildup. See [[tactical-analysis]].

**Pairwise relationships.** On-ball and off-ball EPV between David Silva and each Manchester City teammate, split by direction — distinguishing teammates who *create space for* Silva from those who *benefit from* his passing. The sharpest use of the off-ball component, isolating a relational quantity no aggregate rating could express.

**Defensive style profiling.** Recovery rate against being-attacked rate separates high-press-high-risk from solid-and-contained.

Notably, neither held source produces a season-long off-ball player rating. Doing so runs into the credit-attribution problems above.

## Relation to Other Approaches

| Approach | Off-ball handling |
|---|---|
| [[expected-threat\|xT]], [[vaep]], [[pass-carry-reward\|PCR]] | None — on-ball events only |
| [[martingale-epv\|Martingale EPV]] | Implicit; tracking state includes all players, no per-player credit |
| **Pass EPV surface** | **Positional value, attacking, per player** |
| **[[vdep]]** | **Defensive contribution, team level** |
| **Umemoto & Fujii (2023)** | **Defensive positioning, per player** — cited, not held |
| [[pitch-control]] | Values *space*, not the value of receiving there |
| Spearman (2018), OBSO | Physics-based; the foundation the Fujii off-ball line builds on |

[[pitch-control]] and off-ball value are complementary and often confused. Control asks *who would win the ball here*; off-ball value asks *what would this possession be worth if the ball arrived here*. A player can control empty space of no value, or occupy a dangerous position he would probably lose a contest for.

Note that OBSO now appears twice in this vault under different framings — as "the closest external comparison" on this page's earlier revision, and as the actual substrate of the Fujii group's off-ball and defensive work. Acquiring Spearman (2018) would be worthwhile.

## See Also

- [[probability-surface]] · [[pitch-control]] · [[expected-possession-value]] · [[vdep]]
- [[defensive-valuation]] · [[action-valuation]] · [[counterfactual-simulation]] · [[shap]]
- [[keisuke-fujii]] · [[optical-tracking-data]] · [[event-stream-data]] · [[tactical-analysis]]
- [[action-valuation-frameworks-compared]]
- [[expected-value-possession-framework|Soccer EPV Framework Summary]] · [[football-defence-evaluation-vdep|VDEP Summary]]
