---
title: "Multi-Task Learning"
type: concept
tags: [multi-task-learning, machine-learning, deep-learning, training-technique, representation-learning, regularization]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md, raw/papers/transformer-point-process-football-event-modelling.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 38%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Multi-Task Learning

Multi-task learning trains one model on several objectives simultaneously, typically through a shared representation with task-specific heads and a composite loss:

$$\mathcal{L}_{\text{total}} = \sum_i \lambda_i \mathcal{L}_i$$

The premise is that related tasks provide complementary signal — each acts as a regulariser on the shared representation, discouraging solutions that fit one task by exploiting quirks the others would penalise.

## Why Auxiliary Objectives Change What Is Learned

The clearest motivation in this vault comes from [[scoutgpt]]. A pure next-token objective on football events **optimises for likelihood, which is not the same as tactical value**. Passes are ~67% of events, so a likelihood-optimal model becomes very good at predicting passes and has no incentive to model the rare, decisive actions that determine outcomes.

Adding auxiliary heads that predict goal-scored and goal-conceded probabilities gives the shared representation a reason to encode *consequence*, not just frequency:

$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{gen}} + \mathcal{L}_{\text{aux}}$$

The authors report this improves predictive performance over non-value-aware variants — the auxiliary task helps the primary one, not merely coexisting with it.

## Design Choices That Matter

**Where heads attach.** ScoutGPT activates its value heads **only at outcome-token positions** — the last token of each event block — rather than every token. Attaching a head where its prediction is not meaningful adds noise to the shared representation.

**Loss weighting.** The relative scale of terms is a real hyperparameter. [[nmstpp]] weights its timing RMSE by ×10 to bring it onto the same scale as two cross-entropy terms; [[sig-model]] tunes its $\lambda$ by grid search and finds $\lambda = 1$ best. Mis-weighted losses mean one task silently dominates.

**Task masking.** ScoutGPT *excludes* player-ID prediction from the generative loss, because at inference player identity is injected from the lineup rather than generated. Training the model to predict something it will never be asked to predict wastes capacity and creates a train/inference mismatch.

## Multi-Task vs Chained Prediction

Worth distinguishing two things that both produce composite losses:

- **Multi-task** — several *parallel* heads on a shared representation, each predicting a different quantity. ScoutGPT's generative head plus two value heads.
- **Chained / [[autoregressive-model|autoregressive]] factorisation** — several predictions in *sequence*, each conditioning on the last. [[nmstpp]]'s $f_t \to f_z \to f_m$ decomposition.

NMSTPP's composite loss looks multi-task but is really the latter: the three terms correspond to factors of one joint density, not independent objectives. The ablation confirms it — severing the conditioning links degrades performance, which would not happen if the tasks were genuinely parallel.

## When It Helps and When It Hurts

**Helps** when tasks share structure, when the auxiliary task has more or cleaner labels than the primary, or when the auxiliary task encodes a property you want the representation to have.

**Hurts** through *negative transfer* — tasks with conflicting gradients pull the shared representation apart, and both do worse than they would alone. Common remedies are task-specific layers beneath shared ones, gradient-based loss balancing, or simply separate models.

## Related Patterns in This Vault

- **[[bert]]** trains masked language modelling and next-sentence prediction jointly; the ablation shows removing NSP costs 3.5% on QNLI, so the auxiliary task genuinely helps.
- **[[rlhf]]** adds a KL penalty against the SFT policy — a composite objective, though the second term is a constraint rather than a task.
- **[[variational-autoencoder|VAE]] ELBO** balances reconstruction against a KL term. Again composite, and again the balance matters, as $\beta$-VAE makes explicit.

The common thread is that **adding a term to a loss is a way of specifying what the representation should encode** — a lever distinct from architecture or data.

## See Also

- [[scoutgpt]]
- [[nmstpp]]
- [[bert]]
- [[rlhf]]
- [[autoregressive-model]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
