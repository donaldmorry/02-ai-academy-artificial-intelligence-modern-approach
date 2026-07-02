# Chapter 24: Deep Learning for Natural Language Processing

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24
> **Part:** Part VI: Communicating, Perceiving & Acting
> **Estimated time:** 13–15 hours
> **Prerequisites:** Chapter 21: Deep Learning; Chapter 23: Natural Language Processing

---

## Chapter Overview

Cover modern neural NLP: word embeddings (Word2Vec, GloVe), recurrent and encoder-decoder models, the transformer architecture, self-attention, BERT and pretraining-finetuning, and large language model capabilities and limitations.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Explain distributional semantics and train word embeddings
2. Build sequence models with RNNs, LSTMs, and encoder-decoder architectures
3. Derive scaled dot-product attention and multi-head attention
4. Describe transformer encoder and decoder blocks
5. Apply BERT-style masked language modeling and finetuning
6. Evaluate LLMs on downstream tasks and interpret limitations
7. Discuss prompt engineering and in-context learning at introductory level

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Word Embeddings](./section-01-word-embeddings.md) | Word2Vec, GloVe, subword tokenization |
| 2 | [Neural Language Models](./section-02-neural-language-models.md) | RNN LMs, perplexity improvements |
| 3 | [Seq2Seq and Attention](./section-03-seq2seq-and-attention.md) | Machine translation, alignment, attention |
| 4 | [The Transformer](./section-04-the-transformer.md) | Self-attention, positional encoding, layer norm |
| 5 | [BERT and Pretraining](./section-05-bert-and-pretraining.md) | MLM, NSP, finetuning for classification |
| 6 | [GPT and Autoregressive LMs](./section-06-gpt-and-autoregressive-lms.md) | Causal attention, scaling laws overview |
| 7 | [LLM Capabilities](./section-07-llm-capabilities.md) | Prompting, few-shot, chain-of-thought preview |
| 8 | [NLP Ethics](./section-08-nlp-ethics.md) | Bias in LMs, hallucination, mitigation strategies |

---

## Lab / Project

Finetune a pretrained transformer (e.g., DistilBERT) on a text classification task. Compare with bag-of-words baseline from Chapter 23 concepts.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Applied NLP | [Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md): BERT fine-tuning in Keras |
| Text preprocessing | [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-02-text-preprocessing.md): tokenization foundations |
| Attention mechanisms | [Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md): sequence transduction |
| Deep feedforward theory | [Course 3 Chapter 06](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md): backprop formalism |

---

## Prerequisites

- Chapter 21: Deep Learning
- Chapter 23: Natural Language Processing

---

## Self-Assessment

1. How does self-attention differ from recurrent processing?
2. What are the inputs and outputs of BERT pretraining objectives?
3. Why are positional encodings needed in transformers?
4. What is the difference between encoder-only and decoder-only LMs?
5. Define hallucination in LLMs and one mitigation approach.
6. How does subword tokenization handle rare words?

---

**Previous:** [Chapter 23 — Natural Language Processing](../chapter-23-natural-language-processing/README.md)
**Next:** [Chapter 25 — Computer Vision](../chapter-25-computer-vision/README.md)
