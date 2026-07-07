---
title: "GPT"
type: concept
tags: [transformer, language-modelling, autoregressive-model, deep-learning, stub, needs-review]
sources: []
confidence: 0.45
provenance:
  extracted: 0%
  inferred: 95%
  ambiguous: 5%
lifecycle: draft
created: 2026-07-07
updated: 2026-07-07
---

# GPT

> **Stub — no primary source ingested.** This page exists because [[transformer]] and the operation log reference GPT as downstream impact. Its content is general background knowledge, not extracted from a raw source in this vault. Ingest a primary source (e.g. Radford et al., 2018/2019; Brown et al., 2020) to raise confidence.^[inferred: entire page pending source ingestion]

GPT (Generative Pre-trained Transformer) is a family of [[autoregressive-model|autoregressive]] language models built on the [[transformer]] decoder. Each model is pretrained to predict the next token given all previous tokens, and the same objective scales from small models to very large ones.

## Autoregressive Language Modelling

GPT factorises the probability of a sequence via the chain rule, predicting one token at a time conditioned on the left context only (unlike the bidirectional [[bert|BERT]]). This makes it naturally suited to text generation. Later models in the family demonstrated strong few-shot and zero-shot behaviour, performing tasks from natural-language instructions and a handful of examples without task-specific fine-tuning.

## Significance

GPT models are a central illustration of [[scaling-laws|scaling laws]]: performance improves predictably as model size, data, and compute grow. The line is foundational to modern large language models and to [[chain-of-thought]] prompting, instruction tuning, and [[rlhf|RLHF]].

## See Also

- [[transformer]]
- [[bert]]
- [[autoregressive-model]]
- [[scaling-laws]]
- [[chain-of-thought]]
