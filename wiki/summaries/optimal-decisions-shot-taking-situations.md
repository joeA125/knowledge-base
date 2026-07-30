---
title: "A Strategic Framework for Optimal Decisions in Football 1-vs-1 Shot-Taking Situations (Yeung & Fujii, 2024)"
type: source_summary
tags: [summary, sports-analytics, game-theory, action-valuation, theory-based-modelling, proxy-target, off-ball, pitch-control, deep-learning, evaluation, event-stream-data]
sources: [raw/papers/optimal_football_decisions_shot_taking_situations.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 12%
  ambiguous: 3%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# A Strategic Framework for Optimal Decisions in Football 1-vs-1 Shot-Taking Situations

**Authors:** [[calvin-yeung]] · [[keisuke-fujii]] (senior)
**Venue:** *Complex & Intelligent Systems*, Springer. Received Dec 2023, accepted Apr 2024, published 27 May 2024. Open access.
**Code:** github.com/calvinyeungck/Football-1-vs-1-Shot-Taking-Situations-Analysis

The vault's **first game-theoretic framework**, and its first to compute an *optimal* policy rather than evaluate an observed one.

## What Is New Here

Every other valuation framework in this vault models **one side**. VAEP values the attacker's action; VDEP values the defence; OBSO values a receiver's position. None models the two as *interacting agents choosing simultaneously*.

Yeung & Fujii frame shot-taking as a **static two-player zero-sum game**:

$$S_{shooter} \in \{\text{Shoot}, \text{Pass}\}, \qquad S_{defender} \in \{\text{Blocking}, \text{Not Blocking}\}$$

and solve for [[game-theory|Nash equilibrium]]. See [[game-theory]] for why this matters beyond football.

The paper is explicit about why it chose game theory over reinforcement learning: RL "excels in learning policies and determining optimal decisions" but "often lacks the ability to explicitly explain *why* a specific decision is considered optimal without supplementary manual analysis." Game theory forces both agents' strategies into the model.

## The Payoffs: xSOT and xOSOT

Goals are too rare to serve as payoffs, so the paper substitutes the **minimum requirement of a good shot** — that it be on target. A third instance of [[rare-event-proxy-targets|proxy targets]], explicitly motivated as such.

**xSOT** — Expected Probability of Shot On Target, the payoff for shooting. Modelled by the complement of the other two outcomes, via the law of total probability:

$$xSOT = \mathbb{E}[1 - \min(\hat{P}(S_{off}) + \hat{P}(S_{block}), 1)]$$

with $\hat{P}(S_{off})$ and $\hat{P}(S_{block})$ each an MLP. **xOSOT** — the payoff for passing, being the best available off-ball attacker's xSOT weighted by their probability of controlling the ball. See [[xsot]].

## The Theory-Based Shot Block Model

The paper's methodological centrepiece, and the reason it is more than a game-theory application.

Rather than feeding raw defender coordinates to a network, it builds an **explicit geometric model** of shot blocking and uses that model's *output* as a feature. See [[theory-based-modelling]].

The construction, in five steps: filter to defenders inside a feasible block zone; condition on shot angle $\theta$, assumed uniform across the feasible angle; partition the event space so only one defender can block (defender $d$ gets a chance only if $1 \dots d{-}1$ fail); model each defender with a **truncated normal PDF** over angular displacement, with width growing in distance from the shooter; integrate numerically by the trapezoidal rule.

**Improvements over [[c-obso|C-OBSO's]] block model**, all four stated explicitly:

| Change | Reason |
|---|---|
| **Exclude the goalkeeper** | A saved shot is still *on target* — the target has changed, so the keeper is irrelevant |
| **Continuous rather than discrete angle** | More precise PDF evaluation |
| **Truncated normal, not normal** | A defender's reach is bounded, not infinite |
| **Sequential event space** | If one defender blocks, later ones cannot |

The first is the sharpest: because the payoff is *shot on target* rather than *goal*, the goalkeeper stops being part of the problem. Changing the target changed the model.

## The Ablation That Justifies the Hybrid

Cross-entropy loss on shot-block prediction:

| Model | CEL |
|---|---|
| **MLP + theory-based feature** | **0.4876** |
| ElasticNet | 0.5417 |
| MLP, basic shooter features only | 0.5545 |
| MLP, raw player coordinates (22 × 4 vector) | 0.5684 |
| Historical percentage | 0.5783 |
| XGBoost | 0.6354 |
| CatBoost | 0.7096 |
| *Theory-based model alone* | *0.9220* |

Two findings worth separating.

**The hybrid beats both components.** Theory alone is 0.92, MLP-without-it 0.55, combined 0.49. Neither ingredient is sufficient.

**Handing the network raw coordinates is worse than giving it nothing** — 0.5684 against 0.5545 for shooter features alone. With 2,575 shots, a 93-dimensional player vector is noise. The theory-based model is doing dimensionality reduction that the data cannot support learning.

**Tree ensembles perform worst**, which inverts the [[gradient-boosting|VAEP result]] where CatBoost beat everything. The difference is sample size — 8.5M actions there against 2,575 shots here — and the authors note the trees show clear overfitting.

## The Equilibrium

Across 1,468 filtered shot situations from World Cup 2022:

| | Defender: Blocking | Defender: Not Blocking |
|---|---|---|
| **Shooter: Shoot** | 0.0866 | **0.2508** |
| **Shooter: Pass** | **0.2456** | 0.2481 |

**Nash equilibrium: (Pass, Block).** Neither agent gains by deviating unilaterally.

Two readings:

**Blocking is right, contra the "Liverpool" argument.** The paper's motivation is a coach's claim that declining to block long shots is smart — it tempts low-value shots, gives the keeper sight of the ball, and avoids unpredictable second balls. Against this data, that is not the equilibrium strategy.

**Shooters shoot too much.** When the defender blocks, passing is worth 0.2456 against shooting's 0.0866 — a gap of 0.159. The paper reads this as: under expectation there is usually a better-placed teammate. That is a claim about *systematic* decision error, which is unusual; most valuation work stops at describing what happened.

## Validation

No ground truth for action value exists, so alignment with established metrics is used instead. Across World Cup 2022 teams:

| Correlation | $r$ |
|---|---|
| xG vs average goals | 0.46 |
| **xSOT vs average goals** | **0.58** |
| xG vs xSOT | 0.88 |
| xG vs xOSOT | 0.93 |

**xSOT tracks actual goals better than xG does.** A modest sample (32 teams) and a correlational argument only, but the direction is the interesting part: a metric built on a *denser proxy* outperforms one built on the outcome itself — the same pattern as [[hpus]], [[obso|OBSO]] and [[vdep]].

A chi-square test on consecutive shot outcomes ($p = 0.96$) supports treating shots as independent, which is what licenses the static one-stage game.

## Data

StatsBomb open data: EURO 2020 (51 matches) and World Cup 2022 (64), 2,575 shots — 1,043 off target, 850 on target, 682 blocked. 80/20 split, 5-fold CV.

Notable that this runs on **freely available data**, in a literature where almost everything else needs a commercial licence. The trade-off is that StatsBomb 360 gives positions per event and **no velocities**, which are set to zero — degrading the PPCF computation and forcing the static-game assumption.

## Assessment

**Strengths.** The game-theoretic framing is a genuine addition rather than a reframing; modelling both agents is something no other held source attempts. The theory-based/ML ablation is clean and its lesson generalises well beyond football. Open data and open code. And the authors are candid that game theory was chosen over RL for *explanatory* reasons, not performance.

**Weaknesses.**

- **Two strategies per agent.** "Pass" collapses ten possible recipients into one option; "Block" collapses every defensive posture. The equilibrium is over a heavily coarsened game.
- **Static one-stage**, forced by absent velocity data. Shot-taking is manifestly dynamic — the defender closes as the shooter decides.
- **$MLP_{off}$ barely beats baselines** (0.6696 against 0.6749 for a historical percentage) and its confusion matrix is near-chance (51.96% / 53.11%). The shot-off component is weak, and the authors say so.
- **2,575 shots**, which is why every complex model overfits.
- **Complete information and rationality** are assumed. Players do not observe payoff tables.
- **Aggregate equilibrium only.** The reported solution is over all shooters and defenders pooled; per-team or per-role equilibria are proposed but not computed.
- **No comparison against a decision-making baseline** — no RL policy, no rule-based heuristic. Consistent with the vault-wide benchmarking gap.

## Why It Matters Here

**It is the vault's first optimal-policy result.** [[policy-modelling]], [[reinforcement-learning]] and [[value-iteration]] all state that this literature estimates value under the *observed* policy and does not solve for an optimal one, because the counterfactual would be unfounded. This paper solves for one — by restricting to a two-action game where the counterfactual *is* founded. That correction is now recorded on those pages.

**It supplies a second worked instance of theory-plus-ML hybridisation**, joining [[expected-goals|Fernández et al.'s]] use of an event-data xG as a feature for a tracking-data xG. Two independent groups reaching the same architectural idea is worth noting.

## See Also

- [[game-theory]] · [[xsot]] · [[theory-based-modelling]]
- [[policy-modelling]] · [[reinforcement-learning]] · [[rare-event-proxy-targets]]
- [[expected-goals]] · [[obso]] · [[c-obso]] · [[pitch-control]] · [[action-valuation]]
- [[calvin-yeung]] · [[keisuke-fujii]] · [[nagoya-university]]
- [[action-valuation-frameworks-compared]]
