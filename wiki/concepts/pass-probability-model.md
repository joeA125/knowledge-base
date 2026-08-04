---
title: "Pass Probability Model"
type: concept
tags: [pass-modelling, physical-modelling, theory-based-modelling, sports-analytics, optical-tracking-data, pitch-control, probabilistic-classification, evaluation, action-valuation]
sources: [raw/papers/physics_based_pass_probabilities.md]
confidence: 0.9
provenance:
  extracted: 85%
  inferred: 12%
  generated: 2%
  imported: 1%
  ambiguous: 0%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Pass Probability Model

Predicting whether a pass will succeed, and **who will receive it**, from the game state at the moment the ball is kicked.

[[physics-based-pass-probabilities|Spearman et al. (2017)]] is the vault's instance and the root of a long dependency chain: [[pitch-control|PPCF]] is this model evaluated over the whole pitch, [[obso|OBSO]] is built on that, and the entire [[keisuke-fujii|Fujii group]] off-ball line is built on OBSO.

## The Two Components

Treat each pass as a Bernoulli trial. To receive it, a player must **intercept** the ball's trajectory and then **control** it.

**Time to intercept.** Solve the player's equation of motion under $|\dot{r}| \le v_{max}$, $|\ddot{r}| \le a_{max}$ for the minimum time to reach each point on the trajectory. With $\Delta t \equiv T - t_{int}$, the player intercepts if $\Delta t \ge 0$ — but temporal uncertainty $\sigma$ absorbs "differing speeds, reaction times, and effort levels", deliberately unmodelled:

$$P_{int}(T) = \left[1 + e^{-\pi(T - t_{int})/(\sqrt{3}\sigma)}\right]^{-1}$$

A **logistic** CDF, chosen for heavy tails.

**Time to control.** A player near the ball controls it at rate $\lambda$, giving an exponential distribution $P(t) = 1 - e^{-\lambda t}$. At the fitted $\lambda = 4.30$, ~95% chance of control within one second.

**Combined**, with the term that makes control competitive:

$$\frac{dP_j}{dT}(T) = \Big(1 - \sum_k P_k(T)\Big)P_{int,j}(T)\,\lambda$$

The leading bracket is **zero-sum**: probability mass gained by one player is removed from everyone else. Integrating to $t \to \infty$ gives each player's total reception probability; summing over the passing team gives pass success.

## The Design Requirement That Shapes Everything

The authors state four requirements — probabilistic, data-driven, predictive, smooth — and the third does the most work.

**Predictive** means the model may use *only* information available at the moment of the pass. So the ball's trajectory must be **simulated rather than observed**: drag with constant $C_D = 0.25$, mass 0.42 kg, and the Magnus force ignored, so modelled balls travel in straight lines.

That constraint is what makes **hypothetical passes** evaluable — you can ask what would have happened with a different ball velocity, because the model never needed the real one. Everything downstream that asks a counterfactual question inherits this: [[pitch-control|PPCF]] evaluates an imaginary stationary ball, [[drso|DRSO]] moves a defender, [[c-obso]] substitutes a predicted trajectory.

**A model built to be predictive is a model that can be interrogated.** The alternative — fitting to observed trajectories — would be more accurate and useless for every downstream application here.

## Fitting

Parameters found by maximising the Bernoulli likelihood over a grid, on 5,404 training passes:

$$\sigma = 0.45 \pm 0.01\,\text{(stat)} \pm 0.04\,\text{(syst)}\ \text{s} \qquad \lambda = 4.30 \pm 0.28\,\text{(stat)} \pm 1.1\,\text{(syst)}\ \text{s}^{-1}$$

**Separate statistical and systematic errors** — reported by almost nothing else in this vault, and the basis for the claim that physically-meaningful parameters admit priors from measurement. See [[model-selection]].

Systematic error was estimated by fitting per-game and taking the spread across games, which is a cheap and reusable trick: **the variation in a parameter across independent subsets is an estimate of how much the parameter is really pinned down.**

## Performance

On 5,471 held-out passes: **81% accuracy on the receiving team, 68% on the specific receiver.**

Two details worth keeping.

**Threshold shifting.** Accuracy rises 80.5% → **81.9% by moving the success cutoff from 0.5 to 0.27**, because most passes succeed. See [[class-imbalance-evaluation]] — 0.5 is a convention, not a property of the model.

**Systematic underestimate.** The model predicts 67.9% completion against an actual 78.9%. Attributed to the ignored Magnus force, player tendencies, tracking inaccuracies, or team strategy — none modelled, and the gap reported rather than hidden.

## Why It Is Unusual Here

**It validates against a directly observable quantity.** Who received a pass is a fact in the data. Almost everything else in this vault estimates something unobservable — action value, defensive contribution, off-ball worth — and must fall back on [[predictive-validity]], [[split-half-reliability|reliability]] or external criteria.

That makes this one of very few components in the vault's football coverage whose correctness is established rather than inferred, and it is the basis of the validation asymmetry recorded on [[pitch-control-traditions-compared]].

## What It Generates

| Derived from the model | What it is |
|---|---|
| [[pitch-control\|PPCF]] | Evaluate for an imaginary stationary ball at every location |
| [[receiving-efficiency]] | Expected receptions from the [[poisson-binomial]] against actual |
| Pass value | $V_j = p_j f(x_{suc}) - (1-p_j) f(x_{fail})$ — see [[action-valuation]] |
| Hypothetical passing | Optimise ball velocity by simulated annealing; the vault's earliest prescriptive method |

Four applications from one model, which is the argument for physical modelling over task-specific fitting: **the model is reusable because it describes a mechanism rather than a target.**

## Limitations

- **One team, one season** — 38 Crystal Palace matches, 2015-16.
- **Magnus force ignored**, so curved passes are mismodelled. Plausibly part of the 11-point completion gap.
- **Rolling friction not modelled**; aerodynamic drag applies even after ground contact.
- **No player identity.** $\lambda$ and $\sigma$ are global, so a poor first touch and an excellent one are the same player to the model. Later work adds a goalkeeper multiplier ([[drso|DRSO]]) and a defensive advantage ([[obso|Spearman 2018]]), but neither individuates.
- **Kicking constraints are simplified** in the hypothetical-passing extension, which the authors flag.

## See Also

- [[pitch-control]] · [[obso]] · [[receiving-efficiency]] · [[poisson-binomial]] · [[probability-surface]]
- [[theory-based-modelling]] · [[model-selection]] · [[class-imbalance-evaluation]] · [[predictive-validity]]
- [[c-obso]] · [[drso]] · [[action-valuation]] · [[off-ball-value]] · [[william-spearman]]
- [[physics-based-pass-probabilities|Source Summary]] · [[pitch-control-traditions-compared]]
