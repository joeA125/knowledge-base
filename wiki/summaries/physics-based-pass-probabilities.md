---
title: "Physics-Based Modeling of Pass Probabilities in Soccer (Spearman et al., 2017)"
type: source_summary
tags: [summary, sports-analytics, pitch-control, optical-tracking-data, theory-based-modelling, probability-surface, player-evaluation, off-ball, action-valuation, evaluation]
sources: [raw/papers/physics_based_pass_probabilities.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  generated: 1%
  imported: 0%
  ambiguous: 1%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Physics-Based Modeling of Pass Probabilities in Soccer

**Authors:** [[william-spearman]], Austin Basye, Greg Dick, Ryan Hotovy, Paul Pop — **Hudl**, Lincoln, NE
**Venue:** MIT Sloan Sports Analytics Conference, 2017 Research Papers Competition
**Data:** 38 Crystal Palace matches, 2015-16 Premier League. Tracking at 25 fps plus event data. 10,875 filtered passes across 23 games (5,404 train / 5,471 test).

The foundation beneath [[obso|OBSO]], [[pitch-control|PPCF]] and — through them — the entire [[keisuke-fujii|Fujii group]] off-ball and defensive line. Held last of the chain that depends on it.

## The Parameter Question, Settled

This paper fits, by maximum likelihood over a grid search:

$$\sigma = 0.45 \pm 0.01\,\text{(stat)} \pm 0.04\,\text{(syst)}\ \text{s} \qquad \lambda = 4.30 \pm 0.28\,\text{(stat)} \pm 1.1\,\text{(syst)}\ \text{s}^{-1}$$

**So 0.45 and 4.30 are legitimate published fitted values — from *this* paper.**

The vault inferred exactly this when [[beyond-expected-goals|Spearman (2018)]] was acquired, noting the Fujii-group values "match neither exactly — most likely the 2017 published fit." **That inference was correct**, and the chain is now fully resolved:

| Paper | $\sigma$ | $\lambda$ | Source of values |
|---|---|---|---|
| **This (2017)** | **0.45** | **4.30** | Fitted by MLE on 5,404 Crystal Palace passes |
| [[beyond-expected-goals\|Spearman (2018)]] | **0.54** | **3.99** | Refitted by MAP, with priors of 0.5 and 4.2 |
| [[c-obso\|Teranishi et al.]], [[drso\|Umemoto & Fujii]] | 0.45 | 4.30 | The 2017 values, while citing 2018 |

**The error is bibliographic, not numerical.** The Fujii group uses defensible published parameters and attributes them to the wrong paper. Note also that Spearman's own 2018 priors (0.5, 4.2) are *rounded* versions of his 2017 fits rather than exact carry-forwards.

This substantially softens the vault's earlier framing, which implied the values might be wrong. They are not — they are simply a paper older than the citation suggests. See [[obso]].

Separately: **the goalkeeper multiplier $\lambda = 12.9$ used in [[drso|DRSO]] appears in neither Spearman paper.** It is a Fujii-group addition, distinct from Spearman's 2018 $\kappa = 1.72$ defensive advantage.

## The Model

Two components, both physical.

**Time to intercept.** Solve the player's equation of motion under $|\dot{r}| \le v_{max}$ and $|\ddot{r}| \le a_{max}$ to find the minimum time to reach each point on the ball's trajectory. Temporal uncertainty $\sigma$ absorbs "differing speeds, reaction times, and effort levels of the players" — deliberately unmodelled — via a **logistic** CDF.

**Time to control.** A player near the ball controls it at rate $\lambda$, giving an exponential distribution $P(t) = 1 - e^{-\lambda t}$. At $\lambda = 4.30$, a player has ~95% chance of control within one second.

Combined recursively, with the shared-mass term that makes control zero-sum:

$$\frac{dP_j}{dT}(T) = \Big(1 - \sum_k P_k(T)\Big)P_{int,j}(T)\,\lambda$$

**Ball trajectory** is simulated rather than taken from tracking, so the model can evaluate *hypothetical* passes. Drag with constant $C_D = 0.25$; **the Magnus force is ignored**, so modelled balls travel in straight lines. The authors are explicit that this is a simplification whose error they hope the fitted parameters absorb.

## Results

**81% accuracy predicting the receiving team; 68% predicting the specific receiver**, on held-out data using only the game state at the moment of the pass.

Two details worth keeping:

**Threshold shifting.** Accuracy rises from 80.5% to **81.9% by moving the success cutoff from 0.5 to 0.27**, because most passes succeed. A worked instance of the general point on [[class-imbalance-evaluation]] — 0.5 is a convention, not a property of the model.

**Systematic underestimate.** The model predicts 67.9% completion against an actual 78.9%. Attributed to the Magnus force, player tendencies, tracking inaccuracies, or team strategy — none modelled.

## Four Derived Applications

**Receiving and interception efficiency.** Expected receptions from the Poisson binomial, compared against actual. By position:

| | Total | Receiving | Interception |
|---|---|---|---|
| Defender | 1.06 | 1.04 | **1.13** |
| Midfielder | 1.03 | 1.23 | 0.57 |
| Forward | 0.91 | **1.36** | 0.23 |
| Goalkeeper | 0.65 | 0.80 | 0.48 |

Forwards over-receive relative to difficulty; defenders over-intercept. The authors caution that team strategy confounds credit — a striker may decline an interception he could make.

**Pass value.** $V_j = p_j f(x_{suc}) - (1-p_j) f(x_{fail})$, with a "naïve" state value that is a negative exponential in distance to goal. This is [[action-valuation|action valuation]] in embryo, arrived at independently of the Leuven line.

Correlations with team success: reception efficiency against shots 0.64 and against attacking-third passes 0.70; pass value 0.63 and **0.83**.

**Pitch control.** Evaluate the stationary pass probability for an imaginary ball at every pitch location. **This is the origin of the [[pitch-control|PPCF]] tradition** — introduced here as an *extension* of a passing model, not as a primitive. Applied to corners: defending teams exert 4% less control within 5 m of goal when a goal is scored than when the shot is saved.

**Hypothetical passing.** Simulated annealing finds the ball velocity maximising reception probability for an intended receiver; perturbing it gives $\mu_P + \sigma_P$ ("if well-kicked") and $\mu_P - \sigma_P$ ("if poorly-kicked"). High-high means easy; high-low means feasible but requiring skill.

**This is a prescriptive method, published in 2017** — seven years before [[xsot|Yeung & Fujii]] and six before [[drso|DRSO]], and the vault's earliest instance of the prescription task. The authors note computational cost prevents large-scale application, so no metric is built from it.

## Assessment

**Strengths.** Genuinely predictive by construction — uses only information available at the moment of the pass, which is what enables hypothetical evaluation. Parameters have physical units and stated statistical *and* systematic errors, which almost nothing else in this vault reports. The four derived applications all fall out of one model rather than requiring separate machinery.

**Weaknesses.**

- **One team, one season.** 38 Crystal Palace matches; the model is fitted to and evaluated on a single club's games.
- **Magnus force ignored**, so curved passes are mismodelled — plausibly part of the 11-point completion gap.
- **Rolling friction not modelled**; aerodynamic drag is applied even after ground contact.
- **The pass-value state function is admittedly naïve** — distance to goal alone, with three fitted constants.
- **Hypothetical passing does not scale**, by the authors' own account.
- No [[split-half-reliability|reliability]] figures for the derived player metrics.

## Why It Matters Here

**It settles the parameter chain** that two Fujii-group papers propagate, as above.

**It relocates the origin of pitch control.** The vault treated PPCF as OBSO's control term. It is older than that and more general — a pass-reception model whose spatial evaluation happens to yield a control surface. [[pitch-control]] corrected accordingly.

**It confirms the Hudl affiliation**, independently of the 2018 paper. The vault's earlier "Liverpool FC" attribution was an imported error, now doubly refuted.

**It adds a third prescriptive instance**, and the earliest. See [[action-valuation-frameworks-compared]].

## See Also

- [[pitch-control]] · [[obso]] · [[william-spearman]] · [[theory-based-modelling]] · [[probability-surface]]
- [[c-obso]] · [[drso]] · [[off-ball-value]] · [[action-valuation]] · [[class-imbalance-evaluation]]
- [[beyond-expected-goals|Spearman 2018 Summary]] · [[action-valuation-frameworks-compared]]
