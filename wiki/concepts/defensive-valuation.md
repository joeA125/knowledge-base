---
title: "Defensive Valuation"
type: concept
tags: [defensive-valuation, sports-analytics, action-valuation, off-ball, player-evaluation, evaluation, optical-tracking-data, proxy-target, counterfactual]
sources: [raw/papers/football_defence_evaluation.md, raw/papers/evaluation_creating_scoring_opportunities_trajectory_prediction.md, raw/papers/on-ball-actions-football-xt-vs-vaep.md, raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 40%
  ambiguous: 15%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Defensive Valuation

Quantifying the contribution of preventing goals rather than creating them. Long the least-served problem in this vault's football coverage.

## Why It Is Structurally Harder

**The target is a non-event.** Defensive success is the *absence* of an outcome, and absences are hard to attribute. A defence that concedes nothing may have been excellent, or may have faced nobody.

**The base rate is punishing.** A conceding classifier sees ~0.2% positives. See [[rare-event-proxy-targets]].

**Credit is diffuse.** A defensive success involves the presser, the cover, the marker, and the shape they collectively hold — most of whom never touch the ball.

**The data does not record it.** [[event-stream-data|Event data]] logs actions; pressing, screening and holding a line generate none. Only [[optical-tracking-data|tracking]] fixes this.

## The Symptom Across the Vault

Van Dijk is **81st by [[vaep]], 142nd by [[expected-threat|xT]]**. Two results complicate the reading that this is purely definitional:

- He tops **both** of [[duel-skill-rating|Shelopugin's duel-rating tables]] — the information exists in ordinary event data, unmodelled.
- [[vdep|VAEP's conceding classifier scores F1 = 0.000]] on 45 matches. The defensive half of the vault's most-cited framework is not merely weak but empirically inert at that data scale.

## Five Approaches

| Approach | Idea | Example | Level | Held? |
|---|---|---|---|---|
| Negative half of an attacking model | Value defence as reduced $P(\text{concede})$ | [[vaep]] | Player | Yes |
| Contested-event rating | Rate who wins physical contests | [[duel-skill-rating]] | Player | Yes |
| Frequent proxy prediction | Predict recovery / being attacked | [[vdep]] | **Team** | Yes |
| **Counterfactual positioning** | What would a different position have prevented? | Umemoto & Fujii (2023) | **Player** | **No** |
| Spatial suppression | Model the space a defence denies | [[pitch-control]] | Team, implicit | Yes |

## What VDEP Establishes

[[vdep]] is the vault's first *held* framework where preventing value is the target rather than the negative of an attacking one:

1. **Frequent proxies work.** F1 rises from 0.201/0.000 to 0.522/0.484 by predicting recoveries and effective attacks instead of goals.
2. **Off-ball features carry the signal.** [[shap]] puts nearest-defender distance and nearest-attacker position at the top.
3. **Defensive style is separable.** Recovery rate against being-attacked rate distinguishes high-press-high-risk from solid-and-contained.

What it does not do: **it is team-level only.** Its own proposed next step — compute the change in VDEP when a player moves differently — is not implemented in that paper.

## The Follow-Up Literature

> **Provenance.** [[creating-scoring-opportunities-trajectory-prediction|C-OBSO]] is now **held and read**. The other three works below are **cited only** — bibliographic details verified, but method and capability claims are as stated by their authors and unverified here.

| Work | Contribution | Held? |
|---|---|---|
| Teranishi, Fujii & Takeda (2020), IEEE GCCE pp. 124–125 | Earliest of the line; trajectory-based defensive evaluation | No |
| Umemoto, Tsutsui & Fujii (2022), arXiv:2212.00021 | **GVDEP** — generalises VDEP to player-location level | No |
| **Teranishi, Tsutsui, Takeda & Fujii (2022/23), MLSA** | **[[c-obso]]** — credits movement that creates space for a teammate | **Yes** |
| Umemoto & Fujii (2023), StatsBomb Conference | **Individual defender evaluation** via counterfactual positioning | No |

**Correction, 2026-07-27.** An earlier revision of this page stated individual defensive credit was unaddressed anywhere. That was inferred from VDEP's limitations section without checking the follow-up literature — the mistake other pages here warn about, committed in reverse: treating one paper's stated limitation as the field's state.

### What C-OBSO confirms, now from a primary source

C-OBSO is an *attacking* metric, but it validates the mechanism the defensive work depends on. Its construction — compare the actual world against a predicted reference, attribute the difference to one named agent — is the same [[counterfactual-baseline]] that Umemoto & Fujii apply to defenders.

**The individuating ingredient is the counterfactual, not the data.** [[vdep]] and [[c-obso]] use comparable tracking data; VDEP produces one number per configuration with no way to split it, C-OBSO produces a per-player number. That difference is intervention, not information — which makes the claim that counterfactual positioning individuates defensive credit *mechanically plausible* even though the paper itself is unread here.

It also inherits the mechanism's weakness: C-OBSO is **identically zero under perfect prediction**, so any such metric measures deviation from a particular model's expectation rather than a player property outright.

### The counterfactual positioning method

As described by Fujii in an overview article: identify the highest-[[obso|OBSO]] location, select the closest defender and his grid cell, then search which cell he could have occupied to reduce OBSO most. The output is the positioning that "reduced the threat the most".

Both Fujii counterfactual lines build on [[obso|OBSO]] rather than event classification, placing the group's off-ball work closer to [[probability-surface|Fernández et al.'s]] value surfaces than to VDEP's classifiers.

### Cost

Fujii reports the trajectory method needed **enormous computation to evaluate all 22 players** — one prediction per player evaluated. C-OBSO confirms it: only **three of 22** are predicted. A method whose appeal is modelling interaction is restricted to the smallest interacting subset.

## What Remains Open

- **Individual defensive credit is unverified here.** Addressed in the literature; not in `raw/`.
- **No cross-framework comparison.** GVDEP, counterfactual positioning and VDEP have never been benchmarked against one another, nor against [[vaep]]'s defensive half.
- **Errors of omission** — a defender who fails to press. Counterfactual positioning may cover this in principle, since a better available cell implies the actual one was worse.
- **Reliability unreported** for every defensive metric here — the criterion that matters most for [[recruitment]].
- **Scale.** Full-squad evaluation is prohibitively expensive by the only method that individuates.

## Beyond Football

The structure recurs wherever the valuable outcome is prevented: fraud not committed, outages avoided, infections not transmitted. Same three problems — the target is an absence, the base rate is tiny, credit is shared across a system. Proxy targets and counterfactual attribution are the two standard responses, and this literature uses both.

## See Also

- [[vdep]] · [[c-obso]] · [[counterfactual-baseline]] · [[rare-event-proxy-targets]] · [[class-imbalance-evaluation]]
- [[action-valuation]] · [[vaep]] · [[duel-skill-rating]] · [[off-ball-value]] · [[obso]]
- [[pitch-control]] · [[probability-surface]] · [[keisuke-fujii]] · [[optical-tracking-data]]
- [[action-valuation-frameworks-compared]]
- [[football-defence-evaluation-vdep|VDEP Summary]] · [[creating-scoring-opportunities-trajectory-prediction|C-OBSO Summary]]
