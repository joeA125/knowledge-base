---
title: "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding — Source Summary"
type: source_summary
tags: [deep-learning, transformer, language-modelling, transfer-learning, pre-training, masked-language-model, representation-learning]
sources: [raw/papers/bert-bidirectional-transformers.md]
confidence: 0.95
provenance:
  extracted: 90%
  inferred: 8%
  ambiguous: 2%
lifecycle: reviewed
created: 2026-07-07
updated: 2026-07-07
---

# BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding

**Authors:** Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina Toutanova
**Affiliation:** Google AI Language
**Published:** 2019 (NAACL 2019); arXiv:1810.04805

## Key Contribution

Introduces BERT (Bidirectional Encoder Representations from Transformers), which pre-trains deep bidirectional representations using a [[masked-language-model]] (MLM) objective. Unlike [[language-understanding-gpt|GPT's]] left-to-right pre-training, BERT jointly conditions on both left and right context in all layers. Fine-tuning with one additional output layer achieves SOTA on 11 NLP benchmarks.

## Architecture

BERT uses a multi-layer bidirectional [[transformer]] encoder (not decoder):
- **BERT$_\text{BASE}$:** L=12, H=768, A=12, 110M params (same size as GPT for comparison)
- **BERT$_\text{LARGE}$:** L=24, H=1024, A=16, 340M params

Input representation: WordPiece embeddings (30K vocab) + segment embeddings (sentence A/B) + learned [[positional-encoding|position embeddings]], summed. Special tokens: `[CLS]` for classification output, `[SEP]` for sentence separation.

## Pre-training Tasks

### Task 1: Masked Language Model (MLM)
Randomly masks 15% of input tokens. For masked positions: 80% replaced with `[MASK]`, 10% with a random token, 10% unchanged. The model predicts the original token from bidirectional context. This solves the fundamental problem that standard LMs can only be trained left-to-right or right-to-left, since bidirectional conditioning would let each word "see itself."

### Task 2: Next Sentence Prediction (NSP)
Binary classification: given sentences A and B, predict whether B actually follows A (50%) or is random (50%). Trained using the `[CLS]` representation. Helps with QA and NLI tasks.

Pre-training data: BooksCorpus (800M words) + English Wikipedia (2,500M words). Trained for 1M steps, batch size 256 sequences × 512 tokens = 128K tokens/batch. 4 days on 16/64 TPU chips.

## Key Results

### GLUE Benchmark
BERT$_\text{LARGE}$ achieves **80.5** overall (vs GPT's 72.8, prior SOTA 74.0). On MNLI: 86.7% (vs GPT's 82.1%).

### SQuAD v1.1
BERT$_\text{LARGE}$ ensemble: **93.2** F1 (+1.5 over previous best). Single model with TriviaQA: **91.8** F1.

### SQuAD v2.0 (with unanswerable questions)
BERT$_\text{LARGE}$: **83.1** F1 (+5.1 over previous best).

### SWAG (commonsense inference)
BERT$_\text{LARGE}$: **86.3%** (vs GPT's 78.0%, human expert 85.0%).

## Ablation Findings

1. **Bidirectionality is crucial:** Removing MLM (LTR only, like GPT) drops MRPC by 9.2 points and SQuAD F1 by 10.7 points.
2. **NSP matters:** Removing NSP hurts QNLI (-3.5%) and MNLI (-0.5%).
3. **Model size:** Strict improvements from 3→6→12→24 layers, even on tiny datasets (3.6K examples), confirming that [[scaling-laws|scale benefits transfer to small tasks]].
4. **Feature-based approach:** Concatenating the top 4 hidden layers as fixed features achieves 96.1 F1 on CoNLL NER — only 0.3 behind full fine-tuning.

## Impact

BERT demonstrated that bidirectional pre-training is superior to unidirectional for understanding tasks, complementing GPT's generative strengths. It became the dominant paradigm for NLU (2019–2021), spawning variants (RoBERTa, ALBERT, DeBERTa, DistilBERT) and establishing the [[pre-train-then-fine-tune]] paradigm alongside GPT.

## See Also

- [[masked-language-model]]
- [[pre-train-then-fine-tune]]
- [[transformer]]
- [[language-understanding-gpt|GPT]]
- [[scaling-laws]]
- [[rlhf]]
