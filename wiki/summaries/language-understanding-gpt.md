---
title: "Improving Language Understanding by Generative Pre-Training (GPT) — Source Summary"
type: source_summary
tags: [deep-learning, transformer, language-modelling, transfer-learning, pre-training, autoregressive-model, representation-learning]
sources: [raw/papers/language_understanding_gpt.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-07
updated: 2026-07-07
---

# Improving Language Understanding by Generative Pre-Training (GPT)

**Authors:** Alec Radford, Karthik Narasimhan, Tim Salimans, [[ilya-sutskever]]
**Affiliation:** [[openai]]
**Published:** 2018

## Key Contribution

Introduces the [[pre-train-then-fine-tune]] paradigm for [[transformer]]-based language understanding: a 12-layer decoder-only Transformer is first pre-trained as a left-to-right language model on the BooksCorpus (800M words), then fine-tuned with minimal architectural changes on downstream tasks. Achieves SOTA on 9 of 12 benchmarks, demonstrating that generative pre-training enables effective transfer to discriminative tasks.

## Architecture

A 12-layer decoder-only [[transformer]] with masked [[multi-head-attention]] (768-d states, 12 heads, 3072-d FFN). Uses learned [[positional-encoding|position embeddings]] (not sinusoidal), BPE tokenisation (40K merges), GELU activation, and [[layer-normalization]]. 117M parameters total.

Pre-training objective: standard left-to-right language modelling $L_1(\mathcal{U}) = \sum_i \log P(u_i | u_{i-k}, \ldots, u_{i-1})$.

## Task-Specific Input Transformations

Rather than task-specific architectures, GPT converts all tasks into token sequences with delimiter tokens:
- **Classification:** `[Start] Text [Extract]`
- **Entailment:** `[Start] Premise [Delim] Hypothesis [Extract]`
- **Similarity:** Both orderings processed, representations added element-wise
- **Multiple choice:** Each answer concatenated with context separately

Only one new linear output layer $W_y$ is added per task. An auxiliary LM loss ($\lambda = 0.5$) during fine-tuning improves generalisation and convergence.

## Key Results

| Task | GPT | Previous SOTA | Improvement |
|---|---|---|---|
| Story Cloze (commonsense) | **86.5%** | 77.6% | +8.9% |
| RACE (QA) | **59.0%** | 53.3% | +5.7% |
| MultiNLI (entailment) | **82.1%** | 80.6% | +1.5% |
| GLUE (overall) | **72.8** | 68.9 | +3.9 |

## Key Ablation Findings

- **Pre-training is essential:** Without pre-training, average performance drops 14.8%.
- **Transformer > LSTM:** Replacing the Transformer with a single-layer 2048-unit LSTM causes a 5.6-point average score drop, confirming the Transformer's structured memory aids transfer.
- **Auxiliary LM objective:** Helps on large datasets (NLI, QQP) but not on smaller ones.
- **Layer transfer:** Each additional Transformer layer transferred improves performance, up to the full 12 layers.

## Zero-Shot Behaviour

The pre-trained model exhibits steadily improving zero-shot performance on sentiment analysis, Winograd schemas, linguistic acceptability, and QA as pre-training progresses — evidence that the LM learns task-relevant capabilities without any supervised fine-tuning.

## Impact

GPT established that left-to-right Transformer LMs pre-trained on large text corpora could transfer effectively to diverse NLP tasks with minimal architectural changes. This [[pre-train-then-fine-tune]] paradigm became the foundation for GPT-2, GPT-3, and was refined by [[bert-bidirectional-transformers|BERT]] (which showed bidirectional pre-training via [[masked-language-model]] was superior for many tasks) and later by [[training-lm-follow-instructions-with-human-feedback|InstructGPT/RLHF]].

## See Also

- [[pre-train-then-fine-tune]]
- [[masked-language-model]]
- [[transformer]]
- [[scaling-laws]]
- [[bert-bidirectional-transformers|BERT]]
