---
title: "BERT"
type: concept
tags: [transformer, language-modelling, deep-learning, stub, needs-review]
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

# BERT

> **Stub — no primary source ingested.** This page exists because [[transformer]] and the operation log reference BERT as downstream impact. Its content is general background knowledge, not extracted from a raw source in this vault. Ingest the original paper (Devlin et al., 2018) to raise confidence.^[inferred: entire page pending source ingestion]

BERT (Bidirectional Encoder Representations from Transformers) is a pretrained language representation model built on the [[transformer]] encoder. Unlike left-to-right [[gpt|autoregressive]] models, BERT is trained to condition on both left and right context simultaneously, producing deeply bidirectional representations.

## Pretraining

BERT is pretrained on two self-supervised objectives:

- **Masked language modelling (MLM):** a fraction of input tokens are masked and the model predicts them from surrounding context.
- **Next-sentence prediction (NSP):** the model predicts whether two sentences are consecutive.

The pretrained encoder is then fine-tuned with a small task-specific head for downstream tasks such as classification, question answering, and named-entity recognition.

## Significance

BERT is one of the archetypal demonstrations that a single [[transformer]]-based model, pretrained at scale, can be adapted to many language-modelling and NLP tasks — a paradigm that reshaped the field alongside the [[gpt|GPT]] line of decoder models.

## See Also

- [[transformer]]
- [[gpt]]
- [[attention-mechanism]]
