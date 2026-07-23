---
title: "The Path to a Goal: Understanding Soccer Possessions via Path Signatures — Source Summary"
type: source_summary
tags: [sports-analytics, path-signature, event-prediction, action-valuation, event-stream-data, feature-engineering, predictive-validity, evaluation, deep-learning]
sources: [raw/papers/understanding_football_posessions_using_path_signatures.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-23
updated: 2026-07-23
---

# The Path to a Goal: Understanding Soccer Possessions via Path Signatures

**Authors:** [[david-hirnschall]], [[robert-bajons]] (equal contribution)
**Affiliation:** Institute for Statistics and Mathematics, Vienna University of Economics and Business
**Published:** 2025

## Key Contribution

Two contributions. First, the [[sig-model]] — a next-action predictor that encodes an entire possession as a [[path-signature]], removing the need for both fixed historical windows and handcrafted geometric features. Second, [[lpv]] — a possession value metric built from established xG and xT components rather than arbitrary constants.

It beats the [[seq2event]] transformer benchmark on most losses while running ~2.5× faster, using a plain feedforward network.

## The Argument Against Fixed Windows

The paper's framing critique of [[seq2event]] and [[nmstpp]]: both take a fixed window of the last $k$ actions, which is unnatural for football because

- a fixed window **spans possession boundaries**, mixing in actions from previous possessions — either both teams' actions, or one team's with spatio-temporal discontinuities where the opponent's intervening possession was cut out;
- it **discards possession length entirely**, though length carries tactical information.

The natural unit is the possession itself. But that produces variable-length sequences at irregular sampling frequencies, which recurrent and transformer architectures do not handle natively — hence the appeal of a representation that is inherently length-agnostic.

Supporting evidence (Table B8): sweeping Seq2Event's window over 5, 10, and 40 past actions shows **no clear optimum**, which the authors read as the window being an arbitrary choice rather than a tuned one.

## Path Signatures

The [[path-signature]] of a path is a sequence of iterated integrals that, under mild conditions and appropriate augmentation, characterises the path uniquely up to tree-like equivalence. Properties that matter here:

- handles **variable-length, irregularly-sampled** series natively;
- **preserves order and interaction** of events, which summary statistics (mean, variance, skewness) discard;
- requires **no feature engineering** — geometry is encoded implicitly;
- truncation error decays factorially, $\mathcal{O}(1/M!)$.

The model uses order-3 **log-signatures**, which remove the polynomial redundancy in the full signature via the shuffle product, giving 55 dimensions.

## Architecture

Deliberately simple. Continuous features $(x, y, T)$ → log-signature (55 dims). Action sequence → embedding → weighted average with weights $\propto 1/\text{position}$, favouring recent actions (7 dims). Concatenated with current score advantage (1 dim) → 63 dims → feedforward network, one hidden layer of 256, LeakyReLU($\alpha=0.2$) → 9 outputs (7 action logits + $x$ + $y$).

Loss: $L(\theta) = \text{RMSE}_{(x,y)} + \lambda \, \text{CEL}_{\text{actions}}$, with $\lambda = 1$. Goals, possession changes, and match-end events get zero CEL weight, being contextual rather than stylistic.

**No transformer, no recurrence.** The signature does the sequence encoding, so a feedforward net suffices.

## Results vs Seq2Event

| $n_r$ | Model | Test loss | MSE | CEL | Brier | KL |
|---|---|---|---|---|---|---|
| 3 | **Sig-Model** | **0.2084** | **0.1598** | 0.0486 | **0.7968** | **0.1117** |
| 3 | Seq2Event | 0.2129 | 0.1652 | **0.0477** | 0.8034 | 0.1121 |
| 7 | **Sig-Model** | **0.2134** | **0.1612** | 0.0522 | **0.7881** | **0.1169** |
| 7 | Seq2Event | 0.2152 | 0.1642 | **0.0510** | 0.7998 | 0.1202 |

The pattern holds across all five starting points: Sig-Model wins on total loss, MSE, Brier, and KL; Seq2Event wins narrowly on CEL. So the signature model localises the next action better while predicting its *type* very slightly worse.

Runtime: 195–281s against Seq2Event's 250–688s, a ~2.5× reduction at early forecasting points.

**Note on a prior figure:** Seq2Event's previously reported ~45 minute runtime (repeated in [[nmstpp|Yeung et al.]]) is corrected here — improved PyTorch sample storage and loading brings it down by an order of magnitude. Comparisons citing the old figure overstate the transformer's cost.

## Handcrafted Features Can Hurt

The most transferable finding, and it runs in both directions ([[feature-engineering]]):

- **Sig-Model is *harmed* by adding** Seq2Event's handcrafted geometric features (Table B7) — test loss rises at every $n_r$.
- **Seq2Event is *harmed* by removing* them (Table B8) — restricted to raw $(x, y, T)$, its accuracy drops clearly.

So the features are not universally good or bad; they are a crutch that helps a model lacking an adequate representation and actively interferes with one that already encodes the geometry.

## The KL Divergence Innovation

Alongside CEL and Brier, the paper uses [[kl-divergence]] between predicted and *empirical zone-conditioned* action distributions across 8 hand-defined pitch zones.

The rationale is that CEL and Brier reward picking the correct action, while KL measures **tactical plausibility** — whether the model's distribution matches what football sense expects in that region (more shots in the penalty area, more crosses from wide attacking zones). A model can be accurate while being tactically implausible, and vice versa.

## LPV

[[lpv]] addresses a specific criticism of [[hpus]]: its constants (0, 5, 10) and pitch areas are unexplained, and [[nmstpp|Yeung et al.]] themselves suggest they may be adjusted arbitrarily — which makes the metric hard to interpret and questions its reliability.

LPV instead assigns each action a value from established models evaluated at the *predicted* location:

$$\text{LAV} = \widehat{xG} \cdot P(\text{shot}) + \widehat{xT} \cdot P(\text{dribble, pass, cross}), \qquad \text{LPV} = \sum_i \text{LAV}_i$$

### Validation
| | poss-util | HPUS | LPV | xG | goals |
|---|---|---|---|---|---|
| vs xG | 0.43 | 0.41 | **0.51** | — | 0.68 |
| vs goals | 0.16 | 0.19 | **0.33** | 0.68 | — |
| vs **future** xG | 0.15 | 0.27 | **0.32** | 0.21 | 0.19 |
| vs **future** goals | 0.17 | 0.26 | **0.28** | 0.17 | 0.11 |

The striking row is [[predictive-validity|future performance]]: **HPUS and LPV both predict next-match outcomes better than xG or goals themselves do.** Possession-value metrics carry signal about a team's underlying quality that scoreline outcomes do not.

The authors also introduce comparing *predicted* against *observed* possession value — inserting 1/0 for the action that actually occurred. The gap measures how effectively a team realises its available threat.

## Premier League 2017/18

Aggregated LPV correlates $R \approx 0.896$ with final points and $R \approx 0.934$ with season xG. The stronger correlation with xG is expected, since both measure offensive quality while points also reflect defence — visible in Manchester United (2nd on points, 5th–6th on offensive measures, Mourinho's pragmatic style) and Arsenal (6th on points, offensively strong, defensively poor).

The predicted-vs-observed gap correlates $R \approx -0.876$ with points: better teams realise more of their available possession value. Tottenham stand out as unusually effective by this measure.

## Limitations

- Interevent time is not modelled, unlike [[nmstpp]] — the authors note it would be straightforward to add.
- No weighting function over actions within a possession; they argue [[hpus]]'s $\phi$ is sensible in spirit but insufficiently justified, and that weighting matters more for long possessions.
- Event stream data only; the authors suggest signatures over all 22 players' trajectories from tracking data as the natural extension.

## See Also

- [[path-signature]]
- [[sig-model]]
- [[lpv]]
- [[kl-divergence]]
- [[feature-engineering]]
- [[predictive-validity]]
- [[action-valuation-frameworks-compared]]
