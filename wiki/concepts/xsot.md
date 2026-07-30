---
title: "xSOT and xOSOT"
type: concept
tags: [sports-analytics, action-valuation, proxy-target, game-theory, off-ball, pitch-control, player-evaluation, counterfactual, single-source]
sources: [raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.8
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# xSOT and xOSOT

[[optimal-decisions-shot-taking-situations|Yeung & Fujii's (2024)]] paired metrics for the two options facing a player in a shot-taking situation. They exist to serve as [[game-theory|game-theoretic payoffs]], which shapes their design in ways worth noticing.

## xSOT — Expected Probability of Shot On Target

The payoff for **shooting**. Rather than model shot-on-target directly, it is derived as the complement of the other two outcomes:

$$xSOT = \mathbb{E}[P(S_{on})] \approx \mathbb{E}[1 - \min(\hat{P}(S_{off}) + \hat{P}(S_{block}), 1)]$$

with $\hat{P}(S_{off})$ and $\hat{P}(S_{block})$ each a small MLP over shooter features (role, location, distance and angle to goal) and, for the block model, a [[theory-based-modelling|theory-based shot block probability]].

The outcome space $\{S_{on}, S_{off}, S_{block}\}$ is exhaustive, so the law of total probability applies. The $\min$ guards against the two estimates summing above 1 — a symptom of fitting them independently rather than as a three-way softmax, which would have enforced the constraint by construction.

## xOSOT — Expected Probability of Off-Ball Player Shot On Target

The payoff for **passing**. The best available teammate's xSOT, discounted by their chance of receiving the ball:

$$xOSOT = \max_{a \in A} \mathbb{E}[P(S_{on} \mid Control_a) \cdot P(Control_a)]$$

where $P(Control_a)$ comes from [[pitch-control|PPCF]]. Two design choices differ from [[obso|Spearman's OBSO]], which it adapts:

- **A max over teammates, not a sum over pitch locations.** OBSO integrates a surface; xOSOT picks the single best recipient. Appropriate when the question is "should I pass, and to whom" rather than "what is this possession worth".
- **PPCF integrated to finite $T$, not $\infty$** — where $T$ is the ball's travel time. Control gained *after* the ball would have arrived is useless, so the infinite-horizon version overstates the option's value.

The second is a genuine improvement on the borrowed component and applies to any use of PPCF as a *passing-option* model rather than a general control surface.

## Why Shot-on-Target Rather Than Goal

Goals are too rare to serve as payoffs — the recurring problem. The paper substitutes **the minimum requirement of a good shot**: that it reach the target. A third instance of [[rare-event-proxy-targets|proxy targets]] in this vault, alongside [[vdep]]'s ball recoveries and [[hpus]]'s possession dynamics.

The choice has a consequence the authors exploit: since a save still counts as on target, **the goalkeeper drops out of the problem entirely** and is excluded from the shot-block model. Changing the target changed which players matter.

## Validation

Across World Cup 2022 teams:

| Correlation | $r$ |
|---|---|
| xG vs average goals | 0.46 |
| **xSOT vs average goals** | **0.58** |
| xG vs xSOT | 0.88 |
| xG vs xOSOT | 0.93 |

**xSOT tracks actual goals better than [[expected-goals|xG]] does** on this sample. Only 32 teams and correlational, but the direction matches a pattern the vault now sees repeatedly: a metric built on a *denser proxy* outperforming one built on the outcome itself. See [[predictive-validity]].

The high xG correlations (0.88, 0.93) are doing different work — they establish that xSOT measures something recognisable rather than something new, which is what licenses using it as a payoff.

## Limitations

- **The two component models are uneven.** $MLP_{block}$ is solid (CEL 0.4876, confusion matrix 67% / 67%); $MLP_{off}$ barely beats a historical percentage (0.6696 against 0.6749) with a near-chance confusion matrix (52% / 53%). Since xSOT is the complement of *both*, its precision is limited by the weaker one.
- **Independent binary models, not a three-way classifier.** Hence the $\min$ clamp.
- **Average-player metric.** Like xG, it deliberately does not condition on shooter identity — future work proposes adding skill features.
- **2,575 shots**, from two tournaments.
- **No velocity.** StatsBomb 360 gives positions per event only, so PPCF runs with velocities set to zero, degrading the control estimate inside xOSOT.
- **No [[split-half-reliability|reliability]] figure**, as with every metric in this vault.

## Position

| | [[expected-goals\|xG]] | **xSOT** | **xOSOT** | [[obso\|OBSO]] |
|---|---|---|---|---|
| Target | Goal | **Shot on target** | Shot on target | Goal |
| Whose | Shooter | Shooter | **Best teammate** | Receiver |
| Off-ball | No | No | **Yes** | **Yes** |
| Purpose | Chance quality | **Game payoff** | **Game payoff** | Positional value |

The purpose row is the distinguishing one. xSOT and xOSOT are not primarily player-rating metrics; they exist so a two-by-two payoff table can be filled in, including its **counterfactual cells** — the "not blocking" column is computed by re-running the models with the closest defender removed. A metric designed to be evaluated under interventions is a different object from one designed to summarise a season.

## See Also

- [[game-theory]] · [[theory-based-modelling]] · [[rare-event-proxy-targets]]
- [[expected-goals]] · [[obso]] · [[pitch-control]] · [[c-obso]] · [[action-valuation]]
- [[calvin-yeung]] · [[keisuke-fujii]]
- [[optimal-decisions-shot-taking-situations|Source Summary]]
