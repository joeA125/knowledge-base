---
title: "Bradley-Terry Model"
type: concept
tags: [statistics, paired-comparison, ranking-system, probabilistic-classification, inference]
sources: [raw/papers/epv_control_and_duel_skills_football.md, raw/papers/bayesian-true-skill-rating.md]
confidence: 0.8
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Bradley-Terry Model

The Bradley-Terry model (1952) is the foundational model for **paired comparisons**: given a set of pairwise contest outcomes, infer a latent strength parameter for each competitor. It underlies the logistic form of [[elo-rating-system|Elo]] and, through it, most of the rating systems in this vault.

## The Model

Each competitor $i$ has a positive strength $\pi_i$. The probability that $i$ beats $j$ is that competitor's share of the pair's total strength:

$$P(i \succ j) = \frac{\pi_i}{\pi_i + \pi_j}$$

Reparameterising $\pi_i = e^{\beta_i}$ gives the logistic form, which makes the connection to logistic regression exact:

$$P(i \succ j) = \frac{1}{1 + e^{-(\beta_i - \beta_j)}} = \sigma(\beta_i - \beta_j)$$

Only *differences* in $\beta$ are identified, so the scale needs an arbitrary anchor — the reason rating systems fix an average at 1500 or 1200. Compare [[identifiability]].

Fitting is standard maximum likelihood over observed comparisons, typically by iterative scaling or gradient methods.

## Why It Matters

Bradley-Terry solves the problem that raw win rates cannot: **strength of schedule**. A 70% win rate against weak opposition and a 55% rate against strong opposition may reflect the same ability, and only a model of the opponent can tell them apart. Everything downstream in the rating literature is an elaboration on this insight.

The elaborations address its limitations, each of which is a real gap:

| Limitation | Addressed by |
|---|---|
| No uncertainty — a strength is a point estimate | [[glicko-rating-system\|Glicko]], [[trueskill]] |
| Static — assumes strength does not drift | Glicko's rating deviation; Glicko-2 volatility |
| Two competitors only | [[trueskill]] via [[factor-graph]] inference |
| No draws | Draw-margin extensions; TrueSkill's draw parameter |
| **Symmetry — no positional or contextual advantage** | Advantage terms; see [[duel-skill-rating]] |

## The Symmetry Assumption

The last row is the one that matters for sports beyond chess, and it is easy to miss because it is an assumption about the *contest*, not the competitors.

Bradley-Terry says the only thing distinguishing the two sides is latent strength. Any structural asymmetry — home advantage, playing white, facing your own goal in an aerial duel — is invisible to it, and gets absorbed into the strength estimates of whoever systematically occupies the advantaged role.

The standard fix is an additive term in the linear predictor, $\sigma(\beta_i - \beta_j + a)$. [[duel-skill-rating|Shelopugin's duel ratings]] make $a$ itself a learned function of context rather than a single fitted constant, which is a meaningful generalisation: advantage varies by pitch location, set-piece type, and how many players are nearby.

## Relation to Thurstone

Bradley-Terry is the **logistic** paired-comparison model. Its Gaussian counterpart is the Thurstone Case V model, in which each competitor's performance is drawn from $\mathcal{N}(s_i, \beta^2)$ and the stronger performance wins — the formulation used on the [[elo-rating-system]] page.

The two are close in practice and differ mainly in tail behaviour: the logistic has heavier tails, so upsets between distant competitors are somewhat more likely under Bradley-Terry. Elo's designer originally specified the Gaussian form; the logistic was argued to fit chess data better, and most modern implementations use it.

[[trueskill]] returns to the Gaussian branch, since Gaussian performance distributions are what make its [[factor-graph]] message passing tractable.

## Use in Football

[[garry-gelade]] applied Bradley-Terry to one-versus-one ability, which the [[epv-control-duel-skills-football|EPV paper]] credits as the first duel metric to properly account for opponent strength and — importantly — the first to be **transferable across leagues**, since a shared scale lets players who never meet be compared through chains of common opponents.

## See Also

- [[elo-rating-system]] · [[glicko-rating-system]] · [[trueskill]]
- [[duel-skill-rating]] · [[league-strength-rating]]
- [[identifiability]] · [[bayesian-inference]]
- [[garry-gelade]]
- [[epv-control-duel-skills-football|Source Summary]]
