---
title: "Clustering"
type: concept
tags: [clustering, machine-learning, statistics, mixture-model, model-selection, evaluation, dimensionality-reduction, tactical-analysis]
sources: [raw/papers/football-event-sequences-spatiotemporal-point-process-mixture-model.md, raw/papers/expected_value_possession_framework.md]
confidence: 0.85
provenance:
  extracted: 45%
  inferred: 50%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-27
updated: 2026-07-27
---

# Clustering

Partitioning data into groups without labels. The vault's instances split cleanly into two philosophies that are worth keeping distinct, because they differ in what a cluster *is*.

## Heuristic Versus Model-Based

| | Heuristic | Model-based |
|---|---|---|
| A cluster is | A group minimising a distance criterion | A **component distribution** in a mixture |
| Chosen by | Linkage rule, $k$ | Likelihood, [[model-selection\|BIC]] |
| Gives probabilities | No — hard assignment | **Yes** — posterior membership |
| Example here | [[dynamic-pressure-lines]] | [[football-event-sequences-point-process-mixture\|Possession clustering]] |

The distinction matters practically. A heuristic method always returns *something*, and its output is only as meaningful as the distance function. A [[mixture-model|model-based]] method makes a falsifiable claim about how the data were generated, can be compared across values of $k$ by penalised likelihood, and reports uncertainty about membership.

## The Two Vault Instances

**Complete-linkage on player positions.** [[dynamic-pressure-lines]] clusters the 22 positions into $k = 3$ bands to recover defensive, midfield and forward lines. Complete linkage is the right choice — it produces compact, well-separated groups, which is what a defensive line *is*; single linkage would chain players across the pitch. But $k = 3$ is imposed, so a team defending 5-4-1 and one defending 4-4-2 both get three lines.

**Mixtures of marked spatio-temporal point processes.** [[football-event-sequences-point-process-mixture|Amezouwui et al.]] cluster whole possessions into tactical types — direct counter-attacks through to elaborate positional play — where each component is a full [[point-process]] over event times, locations and types. Fitted by [[expectation-maximization|EM]], with component count chosen by BIC.

The second is the more ambitious: the objects being clustered are *sequences of variable length*, which no distance-based method handles naturally.

## Validation Without Labels

Clustering's central difficulty. Three approaches, and the vault uses all three:

**Simulation with known ground truth.** Generate data from a known partition, recover it, score with [[adjusted-rand-index|ARI]] — chance-corrected, so it does not reward a method for producing many clusters. This is the only route that gives a genuine accuracy number, and it only tests the method under the generative assumptions it already makes.

**Penalised likelihood.** BIC or AIC to choose complexity. Available only for model-based methods. See [[model-selection]].

**Interpretability against domain vocabulary.** Do the recovered clusters correspond to things practitioners already name? Amezouwui et al. lean on this, and it is the weakest of the three — a recognisable cluster may be recognisable because it is obvious.

Note what is *absent*: no vault source validates clusters against an external outcome. Whether a possession's cluster predicts anything — goals, next-match performance, opponent adaptation — is untested, and would be the analogue of [[predictive-validity]] for this task.

## Clustering as Infrastructure

In both vault instances clustering is a **means**, not a deliverable. Pressure lines exist so that line-breaking passes can be defined and valued; possession types exist to characterise how a team plays. Neither paper's contribution is the partition itself.

That is worth noting because it changes what counts as a good clustering: not the tightest groups, but the ones that make a downstream quantity expressible. [[dynamic-pressure-lines]] is defensible despite $k=3$ being arbitrary precisely because the features it enables — which line was broken, how many opponents outplayed — turn out to carry real signal.

## Related Machinery

- [[non-negative-matrix-factorization|NMF]] — used in [[martingale-epv]] for player similarity and spatial bases. A soft, additive alternative to hard partitioning.
- [[identifiability]] — whether a mixture's components are uniquely determined at all; a precondition for interpreting them.
- [[player-embedding]] — [[scoutgpt]]'s learned embeddings separate by position *without* clustering being asked for, which is a stronger result than a successful clustering.

## See Also

- [[mixture-model]] · [[expectation-maximization]] · [[adjusted-rand-index]] · [[model-selection]] · [[identifiability]]
- [[dynamic-pressure-lines]] · [[tactical-analysis]] · [[point-process]] · [[non-negative-matrix-factorization]]
- [[player-embedding]] · [[predictive-validity]]
- [[football-event-sequences-point-process-mixture|Possession Clustering Summary]]
