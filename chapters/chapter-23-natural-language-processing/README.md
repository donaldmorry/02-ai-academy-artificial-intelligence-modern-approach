# Chapter 23: Natural Language Processing

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23
> **Part:** Part VI: Communicating, Perceiving & Acting
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 12: Quantifying Uncertainty; Chapter 19: Learning from Examples

---

## Chapter Overview

Study language from an AI perspective: morphology, syntax, grammars, parsing algorithms, semantic analysis, and classical language models. Build pipelines for tagging, parsing, and n-gram modeling as foundations for modern neural NLP.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Describe levels of linguistic analysis: morphology, syntax, semantics, pragmatics
2. Implement tokenization, stemming, and part-of-speech tagging
3. Use context-free grammars and parse trees for syntactic structure
4. Apply CYK and chart parsing for CFG recognition
5. Build n-gram language models with smoothing (Laplace, backoff)
6. Evaluate language models with perplexity
7. Connect classical NLP pipelines to neural approaches in Chapter 24

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Language Models](./section-01-language-models.md) | N-grams, Markov assumption, perplexity |
| 2 | [Text Processing](./section-02-text-processing.md) | Tokenization, BPE preview, normalization |
| 3 | [Grammar Formalisms](./section-03-grammar-formalisms.md) | CFG, dependency grammar, Chomsky hierarchy |
| 4 | [Parsing](./section-04-parsing.md) | Top-down, bottom-up, CYK algorithm |
| 5 | [Statistical Parsing](./section-05-statistical-parsing.md) | PCFGs, parse tree probabilities |
| 6 | [Semantics](./section-06-semantics.md) | Word sense, compositional semantics, λ-calculus preview |
| 7 | [Pragmatics and Discourse](./section-07-pragmatics-and-discourse.md) | Coreference, speech acts, dialogue |
| 8 | [NLP Pipeline Integration](./section-08-nlp-pipeline-integration.md) | End-to-end classical systems |

---

## Lab / Project

Train a trigram language model with Kneser-Ney smoothing on a text corpus. Implement a CYK parser for a small CFG and visualize parse trees.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Text classification | [Course 1 Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md): bag-of-words and TF-IDF |
| NLP applications | [Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md): practical NLP pipelines |
| Sequence modeling | [Course 3 Chapter 10](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md): RNNs and attention |

---

## Prerequisites

- Chapter 12: Quantifying Uncertainty
- Chapter 19: Learning from Examples

---

## Self-Assessment

1. What is perplexity and how is it interpreted?
2. Distinguish CFG recognition from parsing.
3. Why is smoothing necessary for n-gram models?
4. What is the CYK algorithm's time complexity?
5. Give an example of a syntactic ambiguity.
6. How do PCFGs assign probabilities to parse trees?

---

**Previous:** [Chapter 22 — Reinforcement Learning](../chapter-22-reinforcement-learning/README.md)
**Next:** [Chapter 24 — Deep Learning for Natural Language Processing](../chapter-24-deep-learning-nlp/README.md)
