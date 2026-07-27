---
title: "Defensive Valuation"
type: concept
tags: [defensive-valuation, sports-analytics, action-valuation, off-ball, player-evaluation, evaluation, optical-tracking-data, proxy-target, counterfactual, needs-review]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.75
provenance:
  extracted: 40%
  inferred: 40%
  ambiguous: 20%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Defensive Valuation

Quantifying the contribution of preventing goals rather than creating them. Long the least-served problem in this vault's football coverage, and the one most other frameworks declare as a limitation rather than addressing.

> **Provenance warning.** The [Beyond VDEP](#beyond-vdep-the-follow-up-literature) section below is built from **citation lists and an author's own overview article**, not from primary sources. None of those papers is held in `raw/`. Details of method and capability are reported as claimed by their authors and have not been verified here. Treat that section as a map of what exists, not as vault knowledge.

## Why It Is Structurally Harder

Not merely under-researched — harder for reasons intrinsic to the task.

**The target is a non-event.** Attacking value has a natural positive outcome to point at. Defensive success is the *absence* of one, and absences are hard to attribute. A defence that concedes nothing may have been excellent, or may have faced nobody.

**The base rate is punishing.** Goals are rare, and conceded goals no less so. A classifier predicting conceding sees ~0.2% positives. See [[rare-event-proxy-targets]].

**Credit is diffuse.** An attacking sequence has a passer, a carrier, a finisher. A defensive success involves the presser, the cover, the marker, and the shape they collectively hold — most of whom never touch the ball.

**The data does not record it.** [[event-stream-data|Event data]] logs actions. Pressing, screening, marking and holding a line generate no events. Only [[optical-tracking-data|tracking]] fixes this.

## The Symptom Across the Vault

Every attacking-perspective framework here ranks defenders low, consistently — Virgil van Dijk is **81st by [[vaep]], 142nd by [[expected-threat|xT]]**.

Two results complicate the easy reading that this is purely definitional:

- Van Dijk tops **both** of [[duel-skill-rating|Shelopugin's duel-rating tables]]. The information distinguishing an elite centre-back is present in ordinary event data; the valuation paradigm just does not model those events.
- [[vdep|VAEP's conceding classifier scores F1 = 0.000]] on a 45-match dataset. The defensive half of the vault's most-cited action-valuation framework is not merely weak but empirically inert at that data scale.

## Four Approaches

| Approach | Idea | Example | Level |
|---|---|---|---|
| **Negative half of an attacking model** | Value defence as reduced $P(\text{concede})$ | [[vaep]] | Player |
| **Contested-event rating** | Rate who wins physical contests | [[duel-skill-rating]] | Player |
| **Frequent proxy prediction** | Predict recovery / being attacked | [[vdep]] | **Team** |
| **Counterfactual positioning** | Ask what a different position would have prevented | Umemoto & Fujii (2023) | **Player** — see below |
| **Spatial suppression** | Model the space a defence denies | [[pitch-control]] | Team, implicit |

## What VDEP Establishes

[[vdep]] is the vault's first *held* framework where preventing value is the target rather than the negative of an attacking one. Three things it shows:

1. **Frequent proxies work.** F1 rises from 0.201/0.000 to 0.522/0.484 by predicting recoveries and effective attacks instead of goals.
2. **Off-ball features carry the signal.** [[shap]] puts nearest-defender distance and nearest-attacker position at the top.
3. **Defensive style is separable.** Recovery rate against being-attacked rate distinguishes high-press-high-risk from solid-and-contained.

What it does not do: **it is team-level only**, stated plainly by its authors. Its own proposed next step — compute the change in VDEP when a player moves differently — is not implemented in that paper.

## Beyond VDEP: The Follow-Up Literature

**Correction, 2026-07-27.** This page previously stated that individual defensive credit was unaddressed anywhere. That was wrong. It was inferred from VDEP's own limitations section without checking the subsequent literature — the mistake other pages here warn about when treating citations as a substitute for primary sources.

The Fujii group has continued this line, and at least one paper attacks individual defensive credit directly:

| Work | Claimed contribution | Held? |
|---|---|---|
| Umemoto, Tsutsui & Fujii (2022) — *Location analysis of players in UEFA EURO 2020 and 2022 using generalized valuation of defense by estimating probabilities*, arXiv:2212.00021 | **GVDEP** — a generalisation of VDEP applied at player-location level | No |
| **Umemoto & Fujii (2023)** — *Evaluation of team defense positioning by computing counterfactuals using StatsBomb 360 data*, StatsBomb Conference | **Individual defender evaluation** via counterfactual positioning | No |
| Teranishi, Tsutsui, Takeda & Fujii (2022/23) — *Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction*, MLSA / Springer | Off-ball evaluation via trajectory prediction; credits *sacrificial* movement | No |
| Teranishi, Fujii & Takeda (2020) — *Trajectory prediction with imitation learning reflecting defensive evaluation in team sports*, IEEE GCCE, pp. 124–125 | Earliest of the line; trajectory-based defensive evaluation | No |

### The counterfactual positioning method

As described by Fujii in an overview article, the Umemoto & Fujii (2023) procedure is: identify the location with the highest OBSO (most dangerous), select the closest defender and his grid cell, then search which cell he could have occupied to reduce OBSO most. The output is the positioning that "reduced the threat the most".

This is structurally the [[counterfactual-simulation|counterfactual]] move applied to *position* rather than to actions or lineups — and it individuates, because the intervention is on one named defender's location. It is what [[vdep]]'s authors proposed and did not build.

Two things worth noting. It rests on **OBSO** (off-ball scoring opportunities, Spearman 2018), so the whole Fujii off-ball line is built on a value surface rather than on event classification — closer in spirit to [[probability-surface|Fernández et al.'s]] machinery than to VDEP's. And it uses **StatsBomb 360** data, a partial-tracking format cheaper than full tracking, which matters for whether the method is reproducible outside elite settings.

### The stated combination, and its cost

Fujii describes combining the trajectory work (which credits movement sacrificed for teammates) with the counterfactual positioning work (which evaluates each defender) as a framework covering the movements of almost all players.

He also reports the limiting constraint: the trajectory method required **enormous computation to evaluate all 22 players**, since it needs a separate trajectory prediction per player evaluated. That is a familiar shape — the same cost problem that forced [[martingale-epv|Cervone et al.]] onto 461 processors.

## What Remains Genuinely Open

Narrower than this page previously claimed, but not empty:

- **Nothing in this list is held in `raw/`**, so none of it has been read here. The capability claims are the authors' own.
- **No cross-framework comparison.** GVDEP, counterfactual positioning and VDEP have never been benchmarked against one another, nor against [[vaep]]'s defensive half, on a shared task. Consistent with the vault-wide [[action-valuation-frameworks-compared|benchmarking gap]].
- **Errors of omission** — a defender who fails to press. Counterfactual positioning may address this in principle, since a better available position implies the actual one was worse; whether it does in practice is unverified.
- **Reliability unreported.** No defensive metric anywhere here has a [[split-half-reliability]] figure, which is the criterion that matters for [[recruitment]].

## Beyond Football

The structure recurs wherever the valuable outcome is a prevented one: fraud not committed, outages avoided, infections not transmitted. The same three problems appear — the target is an absence, the base rate is tiny, and credit is shared across a system. Proxy targets and counterfactual attribution are the two standard responses, and this literature uses both.

## See Also

- [[vdep]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[action-valuation]] · [[vaep]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[off-ball-value]] · [[pitch-control]] · [[counterfactual-simulation]] · [[probability-surface]]
- [[keisuke-fujii]] · [[optical-tracking-data]] · [[event-stream-data]]
- [[action-valuation-frameworks-compared]]
- [[football-defence-evaluation-vdep|Source Summary]]
