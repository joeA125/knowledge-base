---
title: "Constrained Decoding"
type: concept
tags: [constrained-decoding, generative-model, sequence-modelling, language-modelling, transformer, tool-use]
sources: [raw/papers/scoutgpt-generative-transformer-football-player-valuation.md]
confidence: 0.85
provenance:
  extracted: 55%
  inferred: 38%
  ambiguous: 7%
lifecycle: reviewed
created: 2026-07-24
updated: 2026-07-24
---

# Constrained Decoding

Constrained decoding restricts an autoregressive model's generation to outputs satisfying validity rules, by masking invalid tokens at each step rather than filtering after the fact:

$$P(s_t \mid s_{<t}) \propto \text{softmax}(z_t + M_t), \qquad M_t(k) = -\infty \text{ if } k \text{ is invalid}$$

Because the mask is applied to the logits before the softmax, invalid tokens receive exactly zero probability and the remaining mass is renormalised over valid options.

## Why Post-Hoc Filtering Is Worse

Two alternatives to masking, both inferior:

- **Generate then reject.** Sample freely, discard invalid outputs, retry. Wasteful, and the rejection rate compounds over long sequences — a 1% per-token violation rate makes a 1,000-token sequence almost certainly invalid.
- **Train harder.** Hope the model learns constraints from data. It largely will, but "largely" is not a guarantee, and rare states are exactly where training data is thin and violations occur.

Masking makes validity **structural rather than probabilistic** — the model cannot produce an invalid token because the token has no probability mass.

## The Football Case

[[scoutgpt]] generates event tokens in a fixed hierarchical order, applying field-dependent masks at each step:

| Field | Constraint source |
|---|---|
| Team | The two teams in the context block |
| Action | VERSA transition validator — which actions may follow which |
| Coordinates | Admissible pitch ranges |
| Local rules | No immediate self-reception; no defensive actions while possession is retained |

The motivation is that unconstrained sampling produces sequences that are *syntactically* valid but physically impossible — a player receiving his own pass, an interception by the team already in possession. These would be plainly wrong to any viewer, and would corrupt any value computed over the sequence.

An **ownership lock** further preserves the acting player across consecutive events when possession is maintained, and **semantic stopping rules** end an episode on a goal or a possession-ending action rather than waiting for a generic EOS token.

## The Same Idea in LLM Systems

Constrained decoding is now standard wherever generated text must be machine-readable:

- **Structured output** — grammar- or schema-constrained generation for JSON, forcing conformance to a schema rather than hoping for it.
- **[[tool-use|Function calling]]** — restricting generation to valid tool names and argument types.
- **Code generation** — masking against a language grammar so output parses.

The football case and the JSON case are the same technique: a validity automaton runs alongside decoding, and its state determines the mask.

## Dependence on Tokenization

Constrained decoding requires knowing *what kind of thing* is being generated at each step. ScoutGPT gets this for free from its fixed 10-token event blocks — position within the block determines the field, so the applicable constraint is unambiguous. See [[tokenization]].

Subword tokenization makes this harder in text: a grammar constraint is defined over characters or symbols, but generation happens over subword pieces that may straddle boundaries. Practical implementations must track which token continuations remain consistent with the grammar, which is a real engineering cost.

## What It Cannot Fix

Masking enforces validity, not quality. A constrained model produces sequences that are all *legal*, which says nothing about whether they are *plausible* — the distribution over legal sequences is still whatever the model learned.

Nor is it free: if the constraints disagree with the training distribution, masking pushes the model off-distribution and can degrade coherence. It works best when the constraints encode rules the data already respects, so masking mostly removes rare errors rather than fighting the model.

## See Also

- [[scoutgpt]]
- [[tokenization]]
- [[large-event-model]]
- [[tool-use]]
- [[scoutgpt-counterfactual-player-valuation|Source Summary]]
