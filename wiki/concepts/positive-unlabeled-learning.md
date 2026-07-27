---
title: "Positive-Unlabeled Learning (Presence-Only Data)"
type: concept
tags: [positive-unlabeled-learning, selection-bias, statistics, machine-learning, evaluation, sports-analytics, recruitment, transfer-prediction]
sources: [raw/papers/epv_control_and_duel_skills_football.md]
confidence: 0.7
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Positive-Unlabeled Learning (Presence-Only Data)

A setting in which the data records where something *did* happen but never where it *did not*. Confirmed positives and an undifferentiated pool of unlabelled cases — some of which are negatives and some of which are unobserved positives.

The canonical example is species distribution modelling in ecology, where field surveys record sightings but absence of a sighting could mean absence of the species or absence of an observer. Ward et al. (2009) give the standard EM treatment.

## Why It Is Not Ordinary Missing Data

The defining feature is that **the labelling mechanism is correlated with the label**. That makes it a species of [[selection-bias]] rather than a nuisance to be imputed away.

Treating unlabelled cases as negatives biases estimates toward pessimism about the positive class; discarding them throws away most of the data. Neither is acceptable, which is why the setting needs its own treatment — typically either modelling the labelling probability explicitly, or reweighting the observed positives by an estimated selection propensity.

## The Football Instance

[[andrei-shelopugin|Shelopugin]] identifies this structure in transfer data, and it is a good illustration of the general shape.

To learn how a player's [[pass-carry-reward|PCR]] changes after a move, you need observed transfers. But transfers are **decisions, not experiments**:

- A player moving from a weaker to a stronger club moved because the buying club saw upside. Observed post-transfer outcomes for upward moves are drawn from the subset the market judged capable of the step up.
- A player moving down usually moved because of decline, injury, or falling out of favour.

So the model learns "upward movers do well" and "downward movers do badly" from a sample where the direction of movement *already encodes a scouting judgement*. Applied to a new player, predictions are **too optimistic for upward moves and too pessimistic for downward ones**.

Adding features like the new club's rating does not fix this. It tells the model a transfer occurred; it does not tell the model that the transfer was selected.

## The Correction, and Its Weakness

The paper applies a shrinkage toward the mean, scaled by the size of the league jump:

$$\Delta ratings = \frac{r(\text{league}_{new}) - r(\text{league}_{old})}{1500}, \qquad PCR_{adj} = PCR \cdot 0.8^{(\Delta ratings + p_l)}$$

where $p_l$ additionally accounts for leakage from players leaving the dataset.

The direction is defensible: bigger jumps get shrunk harder, which is where the optimism bias is largest. The magnitude is not derived from anything. The base $0.8$ and the $1500$ normaliser are asserted, and the author explicitly flags a more principled treatment as future work.

This is worth noting as a pattern rather than a criticism of one paper. The vault's [[player-development-curve|age-curve correction]] has the same character — right direction, heuristic form, no estimator behind it. Selection problems in sports analytics are usually **acknowledged and patched** rather than modelled, because the principled remedies (Heckman-style corrections, EM over the selection mechanism) need instruments or assumptions the data does not supply.

## A Second Selection Layer

The paper stacks a second, related problem on top: the training set requires ≥100 minutes played in *both* seasons, and next-season minutes are unknowable in advance. An auxiliary classifier predicts the probability of clearing that threshold, using contract length from the EA Sports FC dataset among its features.

That model is itself learning from a selected population, and its output feeds the correction above through $p_l$. The layering is honest but compounding — each patch introduces its own bias, and none is validated separately.

## Elsewhere

The pattern recurs wherever outcomes are only observed for units that were chosen:

- **Credit scoring** — repayment is observed only for approved applicants.
- **Recommender systems** — ratings exist only for items users chose to consume.
- **Medical treatment effects** — outcomes observed only under treatments clinicians assigned.
- **Recruitment generally** — job performance is observed only for candidates who were hired.

The football case is unusually clean as an illustration, because the selecting agent (the buying club) is explicitly making a forecast about the same quantity the model is trying to predict.

## See Also

- [[selection-bias]] · [[transfer-performance-prediction]] · [[recruitment]]
- [[expectation-maximization]] · [[counterfactual-simulation]]
- [[player-development-curve]] · [[pass-carry-reward]]
- [[epv-control-duel-skills-football|Source Summary]]
